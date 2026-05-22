# Documents for both humans and coding agents

* [README.md](./README.md)

# Documents for coding agents

* [./.agents/docs/OVERVIEW.md](./.agents/docs/OVERVIEW.md) ... project overview.
* [./.agents/docs/ARCHITECTURE.md](./.agents/docs/ARCHITECTURE.md) ... how the wrapper works internally (cache keys, locking, restore flow).
* [./.agents/docs/JOURNAL.md](./.agents/docs/JOURNAL.md) ... findings, insights, and peer code review history.
* [./.agents/docs/LTM/INDEX.md](./.agents/docs/LTM/INDEX.md) ... long-term memory index for durable project knowledge under `./.agents/docs/LTM/`.
* [./.agents/docs/TODO.md](./.agents/docs/TODO.md) ... open to-do items.

# Rules and protocols

## General

* This crate is a single-binary utility extracted from the Winterbaume project. Keep its scope narrow: a `RUSTC_WRAPPER` that adds path-normalising cross-tree caching on top of `sccache`. Do not pull in additional features without a clear reuse case.

## File Management

* Work summaries belong under `./.agents/docs`, not under `/tmp`.
* Temporary files belong under `./.agents-workspace/tmp`, not under `/tmp`.
* ❌ Never delete user files without permission. Only safe to delete: files YOU created in THIS session under `./.agents-workspace/tmp/`. Assume all pre-existing files belong to the user.

## Building

* Single-crate project, so bare `cargo` is fine. Note: when iterating on the wrapper itself, run with `RUSTC_WRAPPER=` set to empty so cargo doesn't try to use a possibly-broken wrapper to build the wrapper.
* `cargo fmt` before any `cargo check` / `cargo build` / `cargo test` if `sccache` is in use — its source-hash key is sensitive to whitespace.

## Testing

* When fixing a bug, add a regression test that fails before the fix and passes after.

## Local Lint Gate

Before reporting a Rust change as done, run:

```
RUSTC_WRAPPER= cargo clippy --all-targets --all-features -- -D warnings
RUSTC_WRAPPER= cargo fmt -- --check
```

Fix any failures and re-run. Do not declare a change complete with outstanding clippy errors.

## Shell Pitfalls (prezto defaults)

The user's shell uses prezto, which sets aliases and options that break non-interactive scripts:

* ❌ `cp src dst` prompts interactively when `dst` exists (prezto aliases `cp` to `cp -i`). Always `rm -f dst` before `cp`.
* ❌ `cat > file <<'EOF'` and `echo > file` fail with `file exists` when the target exists (prezto enables `NO_CLOBBER`). Workaround: `rm -f file` before writing, or use `tee` / `/bin/cat`.
* ❌ `rm file` prompts for confirmation on some files (prezto aliases `rm` to `rm -i`). Use `rm -f` for non-interactive deletion.

## Git Workflow

* ❌ Never make discretionary commits. Commit only when the user asks.
* Sign every commit with `-S`. `main` is configured to require verified signatures.

## Documentation

* Append new findings to `JOURNAL.md`; do not edit existing entries in place (except via the established consolidation workflow, once one exists here).
* Use **British English** spellings throughout repo-authored documentation (e.g., "behaviour", "colour", "organise", "analyse", "normalise"). Technical programming terms that are conventionally American (e.g., `serialize` in Rust/serde context) are exempt.
* ❌ In repo-authored documentation (`AGENTS.md`, `README.md`, `.agents/docs/**`), never use full-width parentheses (`（` `)`). Use half-width `(` `)` with a half-width space before/after when adjacent to non-whitespace.
* ❌ Same for full-width colons (`：`). Use a half-width colon followed by a half-width space.
