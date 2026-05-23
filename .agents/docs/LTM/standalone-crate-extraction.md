# Standalone Crate Extraction

## Summary

`sccache-wrapper` was extracted from `winterbaume/tools/sccache-wrapper` into this standalone single-binary crate. The repository now owns the wrapper's package metadata, documentation, and durable memory, while Winterbaume remains an external integration that may drift from this copy.

## Key Facts

- The wrapper source moved to the repository root as `src/main.rs` and `src/scoreboard.rs` with no code changes during extraction.
- `Cargo.toml` no longer inherits workspace metadata and now carries explicit dependency versions and publish-oriented metadata.
- `Cargo.lock` is committed because this is a binary crate.
- The initial standalone verification passed `cargo build --release`, `cargo clippy --all-targets -- -D warnings`, and `cargo fmt -- --check`.
- The main pre-publish gaps are still tracked in `TODO.md`: neutralising `WB_*` environment variables, confirming the repository URL, and deciding how Winterbaume consumes the extracted wrapper.
- Wrapper-relevant long-term memory was copied from Winterbaume into `.agents/docs/LTM/sccache-wrapper-cross-worktree-cache.md` and adapted to root-relative paths.
- Winterbaume's original files were left untouched. The two copies can now diverge, so wrapper-internal knowledge should be edited here and Winterbaume integration knowledge should be edited in Winterbaume.

## Details

The extraction kept the runtime implementation stable and focused package-level changes on making the crate self-contained:

- `src/main.rs` and `src/scoreboard.rs` were carried over verbatim.
- `README.md` was copied over and still contains `WB_*` environment variable names that are too project-specific for a general crate.
- The MIT `LICENSE` was added.
- `.gitignore` ignores `target/` only.
- `Cargo.toml` was converted away from workspace inheritance, with explicit versions for `sha2`, `fslock`, `serde`, `serde_json`, and `sysinfo`.
- `publish = false` was removed.
- `repository`, `keywords`, `categories`, and `readme` metadata were added.

The transplanted memory work made this repository self-describing for future agents:

- `.agents/docs/LTM/sccache-wrapper-cross-worktree-cache.md` was copied and path-adapted from Winterbaume.
- The copied LTM document drops `tools/sccache-wrapper/` prefixes because the wrapper now lives at the repository root.
- Bypass and harness sections were reframed as general integration guidance.
- Winterbaume integration references are retained as concrete deployment examples, not as assumptions about this repository.
- `.agents/docs/LTM/INDEX.md` was updated to list the imported topic document.
- User auto-memory was ported to `/Users/moriyoshi/.claude/projects/-Users-moriyoshi-Source-sccache-wrapper/memory/project_sccache_stale_server.md` and `MEMORY.md`.
- The original `originSessionId` from Winterbaume was dropped because it pointed to a session in another project.

## Files

- `Cargo.toml`: standalone crate metadata and explicit dependency versions.
- `Cargo.lock`: committed binary lockfile.
- `src/main.rs`: wrapper implementation, copied from Winterbaume at extraction time.
- `src/scoreboard.rs`: scoreboard implementation, copied from Winterbaume at extraction time.
- `README.md`: user-facing wrapper documentation; still needs project-neutral environment variable names.
- `.agents/docs/LTM/sccache-wrapper-cross-worktree-cache.md`: imported durable implementation knowledge.
- `.agents/docs/TODO.md`: pre-publish and drift follow-ups.

## Test Coverage

Initial extraction was verified with:

```sh
cargo build --release
cargo clippy --all-targets -- -D warnings
cargo fmt -- --check
```

For future wrapper changes, follow the repository lint gate and clear `RUSTC_WRAPPER` while building the wrapper itself:

```sh
RUSTC_WRAPPER= cargo clippy --all-targets --all-features -- -D warnings
RUSTC_WRAPPER= cargo fmt -- --check
```

## Pitfalls

- `WB_*` names are still embedded in code and docs. Generalising them affects `src/main.rs`, `src/scoreboard.rs`, `README.md`, and any Winterbaume integration scripts.
- `Cargo.toml` contains a repository URL that was initially noted as a placeholder. Confirm it before publishing.
- Winterbaume still has its own copy under `tools/sccache-wrapper/`. Until the integration direction is decided, bug fixes may need deliberate propagation between repositories.
