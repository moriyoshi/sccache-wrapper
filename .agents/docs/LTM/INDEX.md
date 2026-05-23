# Long-Term Memory Index

Durable knowledge files live in this directory. Each entry below is a one-line pointer.

## Synthesis Documents

(none yet — synthesis documents accumulate by consolidating topic docs via the `deep-sleep` workflow once enough topics exist.)

## Source Topic Documents

| Document | Summary |
|----------|---------|
| [agent-memory-workflows.md](agent-memory-workflows.md) | Local memory-consolidation skills copied from Winterbaume and adapted for this crate: `good-sleep`, `reconcile-journal-ltm`, `deep-sleep`, and `distill-memories`. |
| [ci-pipeline.md](ci-pipeline.md) | GitHub Actions workflow adapted from Winterbaume for a standalone crate, including the pass-marker cache strategy, retained job graph, dropped jobs, and trigger choices. |
| [sccache-wrapper-cross-worktree-cache.md](sccache-wrapper-cross-worktree-cache.md) | Cache key normalisation, `.cachekey` sidecars for dependency identity, singleflight locking, `--test` harness caching, exit-status replay, anchored extra-filename rewrite, cross-`CARGO_TARGET_DIR` cache-key normalisation via `extern_basename_key`, GC grouping by `(program_kind, crate_name, metadata)`, scoreboard for in-flight visibility, residual-error handling, stale-server recovery, and integration pitfalls. Imported from Winterbaume when this crate was split out. |
| [standalone-crate-extraction.md](standalone-crate-extraction.md) | Extraction of `sccache-wrapper` from Winterbaume into a standalone publishable crate, including package metadata changes, transplanted memory, and drift risks. |
