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

