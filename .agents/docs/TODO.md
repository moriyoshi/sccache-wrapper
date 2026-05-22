# TODO

## Pre-publish (crates.io)

- [ ] **Rename `WB_*` env vars** to a project-neutral prefix (proposal: `SCCACHE_WRAPPER_*`). Affects `src/main.rs`, `src/scoreboard.rs`, `README.md`. Decide on a back-compat policy: drop `WB_*` cleanly, or read both for a transition window. The winterbaume integration scripts will need a matching update either way.
- [ ] **Generalise default cache dir name**. Currently `winterbaume-rustc-cache` in `src/main.rs:731`. Change to `sccache-wrapper-rustc-cache` or similar.
- [ ] **Confirm `repository` URL** in `Cargo.toml`. Currently `https://github.com/moriyoshi/sccache-wrapper` (placeholder).
- [ ] **Add `authors`** in `Cargo.toml` if the eventual maintainer set differs from git history.
- [ ] **CI**: at minimum a GitHub Actions workflow that runs `cargo build`, `cargo clippy -- -D warnings`, `cargo fmt --check` on push.
- [ ] **CHANGELOG.md** with an initial 0.1.0 entry covering the extracted feature set.

## Known limitations to document or fix

- [ ] **Negative caching is off.** Stderr is not path-normalised, so cached failures would replay another worktree's absolute paths in error diagnostics. Either path-normalise stderr or document why negative caching stays off.
- [ ] **No tests in this crate.** The original lived in a workspace where higher-level e2e tests via cargo builds were good enough. A standalone crate ought to grow at least: cache-key hashing unit tests, arg-filter unit tests, end-to-end cache hit/miss with a tiny dummy crate via `assert_cmd`.
- [ ] **Workspace-root detection fallback** (`git rev-parse --show-toplevel`, src/main.rs:101 area) spawns a git subprocess per rustc invocation when `WB_WORKSPACE_ROOT` is unset. Document the perf hit or cache the result in a once-cell.

## Integration with winterbaume

- [ ] Decide whether winterbaume keeps an in-tree vendored copy or switches to a published `cargo install sccache-wrapper` / git dep. Either way, the in-tree copy at `winterbaume/tools/sccache-wrapper/` is now a fork point — drift will accumulate. Pick a direction soon.
