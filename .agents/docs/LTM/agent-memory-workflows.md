# Agent Memory Workflows

## Summary

The repository carries four local memory-consolidation skills copied from Winterbaume and adapted for this standalone wrapper crate. They organise chronological journal notes into durable LTM topics, reconcile journal coverage, consolidate overlapping LTM files, and promote stable knowledge into overview and architecture documents.

## Key Facts

- Skills live under `.agents/skills/`.
- `good-sleep` distils append-only `JOURNAL.md` entries into topic-organised `.agents/docs/LTM/` documents and extracts open follow-ups into `TODO.md`.
- `reconcile-journal-ltm` audits whether journal entries are already covered by LTM or TODO, fills gaps, and is the explicit exception that may remove already-consolidated journal entries.
- `deep-sleep` performs second-stage LTM-to-LTM synthesis while leaving source topic documents intact.
- `distill-memories` promotes durable LTM findings into `.agents/docs/OVERVIEW.md` and `.agents/docs/ARCHITECTURE.md`.
- The ported skills are adapted starting points and need end-to-end exercise in this repository.

## Details

Four skills were copied from `../winterbaume/.agents/skills/`:

- `good-sleep/SKILL.md`: reads `.agents/docs/JOURNAL.md`, extracts to-dos, creates topic documents under `.agents/docs/LTM/`, refreshes `.agents/docs/LTM/INDEX.md`, and appends an LTM consolidation record to the journal.
- `reconcile-journal-ltm/SKILL.md`: checks journal entries against LTM and TODO coverage, runs `good-sleep` for uncovered material, collapses multiple consolidation records into one canonical record, and deletes already-consolidated substantive journal entries.
- `deep-sleep/SKILL.md`: consolidates overlapping LTM topic documents into broader synthesis documents without deleting the source files.
- `distill-memories/SKILL.md`: promotes durable LTM findings into canonical project docs.

The port was intentionally narrowed for this crate:

- `good-sleep` examples now use wrapper topics such as cache-key construction, singleflight locking, and `extern_basename_key` normalisation instead of Winterbaume domain examples.
- `good-sleep` also carries the repository style rule for British English in documentation.
- `reconcile-journal-ltm` keeps its original structure but adds the British English rule and uses half-width punctuation in examples.
- `deep-sleep` no longer has Winterbaume's AWS-services promotion machinery or the Cross-Call Invariant Inventory promotion path. It is now a pure LTM synthesis workflow.
- `deep-sleep` clarifies that removing already-consolidated journal entries belongs to `reconcile-journal-ltm`, not to `deep-sleep`.
- `distill-memories` targets only `OVERVIEW.md` and `ARCHITECTURE.md`; it does not target a separate `QUALITY_GATE.md`.
- `agents/openai.yaml` was not copied because it was a UI binding for a different agent system with no equivalent here.

## Files

- `.agents/skills/good-sleep/SKILL.md`: journal-to-LTM consolidation workflow.
- `.agents/skills/reconcile-journal-ltm/SKILL.md`: journal/LTM/TODO reconciliation and cleanup workflow.
- `.agents/skills/deep-sleep/SKILL.md`: LTM synthesis workflow.
- `.agents/skills/distill-memories/SKILL.md`: promotion from LTM into overview and architecture docs.
- `.agents/docs/JOURNAL.md`: append-only chronological source material.
- `.agents/docs/TODO.md`: open follow-ups extracted from journal entries.
- `.agents/docs/LTM/INDEX.md`: durable memory index.

## Test Coverage

The skills were copied and adapted manually. At the time of porting, none had been exercised end-to-end against this repository.

The first real run of each skill should verify:

- topic clusters match this crate's actual knowledge boundaries
- file paths assume the standalone repository layout
- generated documentation follows British English spelling
- generated documentation avoids full-width colons and parentheses
- `good-sleep` remains idempotent when existing LTM documents already cover older journal entries

## Pitfalls

- Do not assume the ported skills are final. They are a starting point and may need tuning as this repository accumulates more memory.
- `JOURNAL.md` is append-only unless the `reconcile-journal-ltm` workflow is explicitly in use.
- To-dos extracted from journal entries belong in `TODO.md`, not in LTM documents.
- `deep-sleep` should not delete source LTM documents; it creates synthesis documents above them.
