# Finding — duplicate OneNote pages created for the same meeting, tied to earlier corruption events; "Failed" runs may still complete real work

**Found:** 16 August 2026, live at work, following the `OF05a/b/c` empty-value corruption event and the subsequent BadGateway-flagged run.
**Status:** concrete physical evidence gathered; both duplicate pages to be deleted and the meeting recaptured cleanly to verify today's fixes actually resolve the underlying cause.

---

## What was found

Two separate OneNote pages existed for the same meeting ("NH Performance Feedback - David, Simon & Jin Connect"), created by two different capture attempts earlier in today's live-testing session:

1. **`"18 Aug 2026 - NH Performance Feedback - David, Simon & Jin Connect"`** — created by the run that was affected by the `OF05a/b/c` empty-value corruption (logged in `bug-2026-08-16-of05b-silent-empty-value-flowchecker-blind.md`). This run was shown as failing in the Activity trace at the time.
2. **`"NH Performance Feedback - David, Simon and Jin Connect"`** — created by a subsequent attempt, with a visibly different title (no date prefix, "and" instead of "&") — likely reflecting the Topic layer's title-construction logic producing different output on a repeat call, or a different code path being hit on the second attempt.

## Why this matters

**A run that Activity shows as "Failed" may still have completed real, visible work.** The first attempt above was flagged as failed (via the corruption-caused wrong-branch routing), yet it evidently still created a genuine OneNote page. This means "Failed" in the run trace cannot be relied upon as proof that nothing happened — a meaningful caveat for anyone reviewing today's failed runs and assuming they were all fully no-ops.

**This is likely a direct, physical symptom of the `OF05a/b/c` corruption**, not a separate issue — different capture attempts of the same meeting, hitting the flow in different corruption states, produced two different pages instead of one correctly-deduplicated one. This is exactly the kind of real-world consequence the corruption pattern has been causing all week, now caught with concrete before/after evidence rather than only in run traces.

## Action taken

Both duplicate pages will be deleted, and the meeting will be recaptured fresh — now that the `OF05a/b/c` fix and the subsequent 23-value recovery are both confirmed in place — to verify whether a clean recapture produces exactly one correctly-titled page, as intended.

## Recommended note for the Microsoft ticket

Add this as a concrete, tangible example of real-world impact: not just "Flow Checker showed errors," but "the corruption caused a duplicate customer-facing artifact (two OneNote pages for one meeting) that a failed-run status did not reveal." This kind of end-to-end, user-visible consequence is often more persuasive in a support ticket than technical trace detail alone.

---

**Status: evidence captured, cleanup and recapture pending as the next step.**
