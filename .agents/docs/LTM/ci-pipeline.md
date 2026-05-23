# CI Pipeline

## Summary

The GitHub Actions pipeline was adapted from Winterbaume but trimmed for this standalone single-crate project. It preserves the upstream job shape and pass-marker strategy while dropping workspace, example, e2e, DuckDB, and sccache-specific machinery that does not fit this crate.

## Key Facts

- The job graph is `changes` -> `fingerprint` -> `fmt` -> `clippy` -> `test`.
- Downstream jobs keep the `!failure() && !cancelled()` condition so skipped marker-hit jobs do not block dependants.
- Two composite actions remain: `probe-pass-marker` and `save-pass-marker`.
- The composites live under `.github/actions/<name>/action.yml` and centralise the pinned `actions/cache/*@v5.0.5` usage.
- The workflow intentionally does not use sccache. The crate is small, and bootstrapping the wrapper through itself would add avoidable complexity.
- Triggers are `push` on `main`, `workflow_dispatch`, and `workflow_call`. There is no direct `pull_request` trigger.

## Details

The pipeline was factorised from `../winterbaume/.github/workflows/ci.yml` and reduced to the project surface that exists here. The retained shape is:

```text
changes -> fingerprint -> fmt -> clippy -> test
```

The `fingerprint` job computes a hash and probes per-job pass markers. Formatting, linting, and test jobs can then skip work if their marker is already present. The downstream condition uses `!failure() && !cancelled()` so a skipped upstream job still allows dependent jobs to run when appropriate.

Three repeated patterns were considered for composite actions:

- `setup-rust`: deleted before landing because, after removing sccache, it would only wrap `actions-rust-lang/setup-rust-toolchain`.
- `probe-pass-marker`: kept because the `fingerprint` job has three call sites.
- `save-pass-marker`: kept because each working job saves a marker at the tail.

The following Winterbaume-specific pieces were dropped:

- sccache setup: `SCCACHE_GHA_ENABLED`, `RUSTC_WRAPPER=sccache`, `CARGO_INCREMENTAL=0`, and `mozilla-actions/sccache-action`.
- The `examples` job.
- The Terraform-oriented `e2e` job.
- The `setup-duckdb` composite action.
- DuckDB-related second-pass `cargo clippy --no-default-features` and `cargo test --no-default-features` invocations.
- Workspace-aware cargo flags such as `--workspace --exclude winterbaume-sqlengine-duckdb`.

The remaining cargo commands are plain single-crate commands:

```sh
cargo clippy --all-targets --all-features -- -D warnings
cargo test --no-fail-fast
```

The paths-filter and `hashFiles` inputs were reduced to paths that exist or are plausible here:

- `src/**`
- `tests/**`
- `Cargo.toml`
- `Cargo.lock`
- `rustfmt.toml`
- `.github/workflows/ci.yml`
- `.github/actions/**`

`hashFiles` tolerates missing patterns, so listing future paths such as `tests/**` and `rustfmt.toml` is safe.

## Files

- `.github/workflows/ci.yml`: main CI workflow.
- `.github/actions/probe-pass-marker/action.yml`: cache lookup-only pass-marker probe.
- `.github/actions/save-pass-marker/action.yml`: pass-marker save helper.
- `Cargo.toml` and `Cargo.lock`: package inputs included in fingerprinting.
- `src/**`: source inputs included in fingerprinting.

## Test Coverage

The CI workflow runs formatting, linting, and tests through:

```sh
cargo fmt -- --check
cargo clippy --all-targets --all-features -- -D warnings
cargo test --no-fail-fast
```

For local Rust changes, this repository's stricter local gate still requires clearing `RUSTC_WRAPPER`:

```sh
RUSTC_WRAPPER= cargo clippy --all-targets --all-features -- -D warnings
RUSTC_WRAPPER= cargo fmt -- --check
```

## Pitfalls

- Adding a `pull_request` trigger would be a behavioural change from the Winterbaume-derived setup. Keep that intentional if changed.
- Reintroducing sccache in CI is not automatically useful for this crate. If remote caching is revisited, avoid compiling the wrapper through a possibly broken wrapper binary.
- Keep path filters aligned with real project inputs. Missing globs are tolerated, but irrelevant globs reduce the usefulness of fingerprinting.
