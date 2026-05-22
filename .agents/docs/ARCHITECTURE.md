# Architecture

## Pipeline

```
cargo → sccache-wrapper (RUSTC_WRAPPER) → sccache → rustc
```

1. **Parse** rustc args minimally — extract crate name, output dir, emit flags, source file, `--test` flag, crate type.
2. **Classify cacheability**. Non-cacheable invocations (`--print`, `-vV`, binary/proc-macro crate types) `exec()` straight to `sccache` with zero overhead.
3. **For cacheable invocations**: normalise paths, compute cache key, check cache.
   - **HIT**: hardlink artefacts from cache into `--out-dir`, replay captured stdout/stderr, exit with the recorded rustc exit status.
   - **MISS**: acquire a per-key `flock` (singleflight). If already held, block until the leader finishes, then re-check cache. Otherwise delegate to `sccache`, capture outputs, hardlink artefacts into cache.

## Cache-key construction

The cache key is `sha256(normalised_args || source_content || compiler_identity || dependency_cachekeys)`.

### Filter rules

| Rust arg | Treatment | Reason |
|---|---|---|
| `-C metadata=…` | **keep** | Distinguishes host/target builds — must not collide. |
| `-C extra-filename=…` | **drop** | Filename suffix; restore path remaps it. |
| `-C incremental=…` | **strip before sccache** | Incremental compilation is incompatible with deterministic outputs. |
| `--out-dir`, `-L` | **drop** | Workspace-local; restore path uses current `--out-dir`. |
| `--extern <name>=<path>` | **drop path, keep name + dep `.cachekey`** | Path is worktree-specific; identity comes from the sidecar. |
| `--diagnostic-width`, `--color` | **drop** | Display-only. |
| Other args | **normalise** `$WB_WORKSPACE_ROOT` → `@@WORKSPACE@@`, keep verbatim | |

### Dependency sidecars

Every cached artefact gets a sibling `.cachekey` file in `--out-dir`. When a parent crate is compiled, its `--extern` references resolve to artefacts that carry sidecars; the wrapper reads those values into the parent's cache key. This makes dependency identity content-derived (rather than path-derived), so a changed dep correctly invalidates parents.

An earlier design that hashed only the `--extern` crate name was unsound — parent crates would restore when a dep's identity had actually changed, causing `E0460`/`E0463` link errors.

## Singleflight

Per-key `flock` file lives at `$WB_RUSTC_CACHE_DIR/locks/<hex2>/<sha256>.lock`. The wrapper:

1. Tries `try_lock_exclusive()`.
2. If held, blocks on `lock_exclusive()`, then re-checks the cache (the leader has now finished).
3. Otherwise runs the build, populates cache, releases the lock.

Followers always re-check cache after wait. Concurrent leaders never happen because the lock is exclusive.

## Restore flow

Hardlink each manifest entry from the cache dir into `--out-dir`, mapping the stored `extra-filename` to the current build's `extra-filename`. Hardlinks preserve the executable bit, so test-harness binaries restore as executables. Replay captured stdout/stderr verbatim. Exit with the rustc exit status recorded in `manifest`.

## Manifest format

```
$WB_RUSTC_CACHE_DIR/<hex2>/<sha256>/
  manifest          # extra-filename + file list + exit:N
  args_received     # original rustc command line (normalised)
  args_emitted      # command line sent to sccache (normalised, no -C incremental)
  stdout            # captured stdout
  stderr            # captured stderr
  libfoo-abc123.rlib    # hardlinked artefact (cache and target dir share the inode)
  libfoo-abc123.rmeta
  libfoo-abc123.rlib.cachekey   # sidecar dependents read
  foo-abc123.d      # dep-info with @@WORKSPACE@@ placeholders
```

## Test-harness builds

Cargo invokes rustc with both `--test` and `--crate-type lib`. The emitted binary is `<name><ef>` (no `lib` prefix, no `.rlib`/`.rmeta` extension; `.exe` on Windows). The wrapper detects `--test` and treats the link output accordingly. GC keying uses `(program_kind + "-test", crate_name, metadata)` so test units and lib units never share a bucket even if `-Cmetadata=` ever collides.

## Scoreboard

When `WB_SCCACHE_WRAPPER_SCOREBOARD` is set, every cache-miss leader and waiter registers itself in a per-key JSON file under that directory. A background thread refreshes a heartbeat every couple of seconds. Used to observe stuck builds across concurrent agent sessions. Stale entries (no heartbeat for ~30 s) and finished entries older than five minutes are pruned opportunistically by the next session that touches the file.

See `src/scoreboard.rs` for the on-disk schema.

## Exit-status replay

The manifest records `exit:N`. Cache restore returns that code instead of unconditionally 0. The `if exit_code == 0` gate around `compile_and_cache` is retained for now — stderr is not yet path-normalised, so caching a failure would replay another worktree's paths in error diagnostics. Until that's fixed, only successful builds are stored.
