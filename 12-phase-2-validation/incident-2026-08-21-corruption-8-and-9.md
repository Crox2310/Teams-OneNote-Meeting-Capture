# Incident log — corruption Incident 8 & 9, during #1 build (21 August 2026)

## Summary
While adding the `OccurrenceDate` field to Flow B's trigger (part of the #1 per-occurrence-pages
build), the same mass value-blanking corruption pattern struck **twice** in quick succession —
the same 26 `SetVariable`/Compose actions losing their `value` field each time, matching the
signature already logged 7 times prior to this session.

## Timeline
- Added `OccurrenceDate` as a new optional trigger input (`text_5`), saved draft.
- Flow Checker returned 26 errors — same actions as every prior corruption incident this project
  (`varTargetSectionPagesUrl`, `varOneNoteResolverResult`, `varOutputPageLink`,
  `varOutputPageSelfUrl`, `varPageAction`, `varOutStatus`, and the OneOff-branch equivalents).
- David restarted Flow B (reopened/reloaded) rather than editing through the corruption.
- Re-checked `text_5` and `Compose_UpdateHtmlFragment` (the #2 fix) after restart — **both intact**,
  confirming the restart only lost the same 26 corrupted values, nothing else.
- Restored all 26 values from `known-good-values-master-reference.md`.
- **Corruption struck again**, same 26 actions, before the restore could be verified clean.
- Restored a second time from the same reference. Flow Checker clean. **Published successfully.**

## Significance
This is now **8th and 9th occurrences** of the mass value-blanking pattern (per the running count
in `CURRENT-STATE.md` / the Microsoft ticket draft). Two occurrences in one short editing window
is a new data point — the corruption is not a rare event but something that can recur repeatedly
within a single session, which strengthens the case for prioritising submission of the Microsoft
support ticket. This session's recovery (twice, back to back) was fast only because
`known-good-values-master-reference.md` existed as a ready restore sheet — without it, this would
have cost significant time re-deriving 26 expressions twice over.

## Recommendation
- Escalate priority on submitting the drafted Microsoft ticket (`MICROSOFT-SUPPORT-TICKET-DRAFT-2026-08-15.md`) —
  add this incident (two occurrences in one session) as further evidence of frequency/severity.
- No flow logic changes were needed to recover — purely a restore-from-reference exercise, which
  the master reference sheet is explicitly designed for. Continue keeping it current.

---
*Logged 21 August 2026 during the #1 (per-occurrence recurring pages) build session.*
