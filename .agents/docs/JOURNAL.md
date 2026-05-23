# Journal

Chronological findings, review notes, and decisions. Append at the bottom; do not edit existing entries in place.

## 2026-05-23 — Extracted from winterbaume

`sccache-wrapper` extracted from `winterbaume/tools/sccache-wrapper` into a standalone publishable crate.

What carried over verbatim:

- `src/main.rs`, `src/scoreboard.rs` — no code changes.
- `README.md` — still references `WB_*` env vars (see TODO).
- MIT LICENSE.

What changed:

- `Cargo.toml` is now standalone (no workspace inheritance). Inlined explicit versions for `sha2`, `fslock`, `serde`, `serde_json`, `sysinfo`. Removed `publish = false`. Added `repository`, `keywords`, `categories`, `readme`.
- Added LICENSE, `.gitignore` (target/ only; `Cargo.lock` committed for the binary).

Verified: `cargo build --release`, `cargo clippy --all-targets -- -D warnings`, `cargo fmt -- --check` all pass.

Open follow-ups captured in `TODO.md`:

- `WB_*` env var prefix is project-specific and should be generalised before crates.io publish.
- The winterbaume repo still has its own copy under `tools/sccache-wrapper/`. Removing it requires updating the build wrapper scripts (`.agents/bin/cargo.sh`, `cargo.ps1`, `mise.toml`, `check-build-cache.sh`).
- `repository` URL in `Cargo.toml` is a placeholder — confirm before publishing.

## 2026-05-23 — Transplanted sccache-wrapper memories from winterbaume

Brought across the wrapper-relevant durable knowledge so this repo carries its own history:

- `.agents/docs/LTM/sccache-wrapper-cross-worktree-cache.md` — full topic doc copied and path-adapted. Drops `tools/sccache-wrapper/` prefixes ( the wrapper now lives at the repo root ), reframes the bypass / harness sections to read as integration guidance rather than winterbaume-specific scaffolding, and keeps the Winterbaume integration references as concrete examples of one deployment.
- `.agents/docs/LTM/INDEX.md` updated to list it.
- User auto-memory ported: `/Users/moriyoshi/.claude/projects/-Users-moriyoshi-Source-sccache-wrapper/memory/project_sccache_stale_server.md` + `MEMORY.md`. The `originSessionId` field from the winterbaume copy was dropped — it points to a session in a different project.

Originals in winterbaume are untouched. Two copies will accumulate drift; the wrapper-internal lessons should be edited here from now on, and any winterbaume-integration lessons should be edited there.

## 2026-05-23 — CI pipeline adapted from winterbaume

Set up `.github/workflows/ci.yml` by factorising winterbaume's pipeline (`../winterbaume/.github/workflows/ci.yml`) and trimming it to fit a single-crate project.

Job graph kept identical in shape: `changes` (paths-filter) -> `fingerprint` (hash + per-job pass-marker probes) -> `fmt` -> `clippy` -> `test`. The `!failure() && !cancelled()` chain on downstream jobs is preserved so a `skipped` upstream (i.e. marker hit) does not block dependents.

Factorisation. Three repeated patterns were candidates for composite actions:

- `setup-rust` (toolchain install + optional sccache) — deleted before landing. Once sccache was removed (see below), it would have been a one-step pass-through to `actions-rust-lang/setup-rust-toolchain`, so each job invokes that directly.
- `probe-pass-marker` (`actions/cache/restore` with `lookup-only`, re-exporting `cache-hit` as `hit`) — kept. Three call sites in the `fingerprint` job.
- `save-pass-marker` (`mkdir` + `cache/save`) — kept. Three call sites at the tail of each working job.

Both surviving composites live under `.github/actions/<name>/action.yml` and centralise the pinned SHA of `actions/cache/*@v5.0.5`.

Dropped from winterbaume's pipeline:

- sccache (`SCCACHE_GHA_ENABLED`, `RUSTC_WRAPPER=sccache`, `CARGO_INCREMENTAL=0`, the `mozilla-actions/sccache-action` steps). Project is too small for sccache to pay for itself; using vanilla sccache would also have been the right call to avoid bootstrapping the wrapper with itself, but the simpler answer is to use no remote cache at all.
- `examples` job (no examples in this crate).
- `e2e` job (no Terraform tests).
- The `setup-duckdb` composite action and the duckdb-related second-pass invocations of `cargo clippy --no-default-features` / `cargo test --no-default-features`.
- Workspace-aware `cargo` flags (`--workspace --exclude winterbaume-sqlengine-duckdb`) collapsed to plain `cargo clippy --all-targets --all-features -- -D warnings` and `cargo test --no-fail-fast`.

Paths-filter and `hashFiles` globs reduced to what exists or is plausible here: `src/**`, `tests/**`, `Cargo.toml`, `Cargo.lock`, `rustfmt.toml`, `.github/workflows/ci.yml`, `.github/actions/**`. `hashFiles` tolerates missing patterns, so it is safe to list paths that do not yet exist (`tests/**`, `rustfmt.toml`).

Triggers kept identical to winterbaume: `push` on `main`, `workflow_dispatch`, `workflow_call`. No `pull_request` trigger — matches the upstream choice, presumably so the workflow can be invoked from a release/aggregator workflow.

## 2026-05-23 — Ported memory-consolidation skills from winterbaume

Brought across four skills from `../winterbaume/.agents/skills/` into `.agents/skills/`:

- `good-sleep/SKILL.md` — distil append-only `JOURNAL.md` entries into topic-organised `.agents/docs/LTM/` documents, extract to-dos into `TODO.md`, refresh `LTM/INDEX.md`, and append a `## LTM Consolidation Record` table to the journal.
- `reconcile-journal-ltm/SKILL.md` — audit whether journal entries are already covered by LTM/TODO, run `good-sleep` for the gaps, collapse multiple consolidation-record sections into one canonical record, and delete already-consolidated substantive entries. Explicit exception to the journal's append-only rule.
- `deep-sleep/SKILL.md` — second-stage consolidation that merges overlapping LTM topic docs into synthesis docs while leaving sources intact.
- `distill-memories/SKILL.md` — promote durable LTM findings into `OVERVIEW.md` and `ARCHITECTURE.md`.

Adaptations vs. the winterbaume originals:

- **good-sleep**: replaced winterbaume-specific topic examples ("address parser", "Tonosho-cho kou/otsu") with sccache-wrapper-relevant ones ("cache-key construction", "singleflight locking", `extern_basename_key`). Added a British-English style rule alongside the existing half-width-parens/colons rules per `CLAUDE.md`.
- **reconcile-journal-ltm**: kept verbatim in structure; updated style rule to also call out British English. Converted a few full-width-style example parentheses (`( a )`, `( b )`) to half-width with proper spacing.
- **deep-sleep**: stripped the AWS-services machinery — original Step 5 (promotion into `.agents/docs/services/<service>.md` with the reference-summary / full-distillation modes) and Step 5b (Cross-Call Invariant Inventory promotion bound to the Winterbaume `write-tests` skill and `quality-gate` §2) had no analogue here. The skill is now a pure LTM-to-synthesis consolidator. Also clarified that removing already-consolidated journal entries is `reconcile-journal-ltm`'s job, not `deep-sleep`'s.
- **distill-memories**: dropped the third target document `QUALITY_GATE.md` — this project keeps quality and lint rules in `CLAUDE.md` (the "Local Lint Gate" section) rather than a dedicated doc. Targets are now just `OVERVIEW.md` and `ARCHITECTURE.md`. Did not bring over the `agents/openai.yaml` sub-config file (a UI binding for a different agent system that has no equivalent here).

Caveat: the four skills come unverified. None has been exercised end-to-end against this repo yet; the first real `good-sleep` / `reconcile-journal-ltm` run is likely to surface tuning needs (topic-cluster examples, file-path assumptions). The skills as ported should be treated as a starting point, not a final form.

---

## LTM Consolidation Record

The following sections have been consolidated into long-term memory documents under `.agents/docs/LTM/`:

| Section | LTM Document |
|---------|--------------|
| 2026-05-23 — Extracted from winterbaume | `standalone-crate-extraction.md` |
| 2026-05-23 — Transplanted sccache-wrapper memories from winterbaume | `standalone-crate-extraction.md` |
| 2026-05-23 — CI pipeline adapted from winterbaume | `ci-pipeline.md` |
| 2026-05-23 — Ported memory-consolidation skills from winterbaume | `agent-memory-workflows.md` |

See `.agents/docs/LTM/INDEX.md` for the full index.
