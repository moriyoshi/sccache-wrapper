# sccache-wrapper Project Overview

`sccache-wrapper` is a single-binary `RUSTC_WRAPPER` that sits between cargo and `sccache` to provide cross-worktree Rust compilation caching. It exists because `sccache`'s own `SCCACHE_BASEDIRS` does not normalise paths for Rust compilations, so identical builds in different git worktrees always miss each other's cache.

The wrapper extracts a path-normalised cache key from the rustc command line, hashes it together with the source-tree content and compiler identity, and either restores cached artefacts (via filesystem hardlinks, zero-copy) or delegates the build to `sccache` and stores the result.

## Scope

- A binary CLI (`sccache-wrapper`) intended to be set as `RUSTC_WRAPPER`.
- Optional scoreboard subsystem for cross-session build observability (`--show-scoreboard`).
- Diagnostic subcommands (`--dump-cache`) for inspecting cache state.
- No library API surface. The crate publishes a binary only.

## Origin

Extracted in 2026-05 from `winterbaume/tools/sccache-wrapper`, where it was developed to support concurrent coding-agent worktrees sharing one Rust build cache. The original integration lives there and continues to consume this same source code; the public crate is the canonical home going forward.

## Boundaries

- Not a replacement for `sccache`. The wrapper delegates compilation to `sccache`, which in turn delegates to `rustc`. Removing `sccache` removes the same-tree rebuild cache layer.
- Not a build system. It does nothing for non-rustc invocations (`build.rs` outputs, linker calls, proc macros that shell out to other compilers, etc.).
- Negative caching (caching failed builds) is intentionally off because stderr is not yet path-normalised — replaying it from cache would surface another worktree's paths in error diagnostics.

## Documentation Map

- `README.md`: user-facing setup, env vars, usage, diagnostics, scoreboard.
- `.agents/docs/ARCHITECTURE.md`: internals — cache-key construction, filter rules, singleflight, restore flow.
- `.agents/docs/JOURNAL.md`: chronological findings and review history.
- `.agents/docs/LTM/INDEX.md`: durable long-term memory.
- `.agents/docs/TODO.md`: open backlog.
