# Plan for evening session — 22 August 2026

**Written:** end of afternoon session, for the evening continuation.

---

## Goal: close out UJ3, UJ4, UJ5

All three field-reported issues are closed. Core feature is working. This session targets the remaining user journey gaps — edge cases and reliability improvements, none blocking normal usage but worth closing out properly.

**Model/effort:** Sonnet 4.6, Standard effort for most. Switch to High if any UJ design turns out to be genuinely ambiguous.

---

## UJ3 — Stale-row / duplicate-row detection (1 gap)

**The problem:** if a mapping row exists in `RecurringMeetingSectionMap` but its `PageSelfUrl` points at a deleted, moved, or otherwise invalid page, the flow takes the `PAGE_EXISTS` path and tries to update a page that no longer exists. The `Update_page_content_Existing_Branch` call will fail, but the failure is silent from the user's perspective (the flow may still report success via `OutStatus`).

**Design needed before building:** what should the flow do when `PAGE_EXISTS` is decided but the subsequent update fails? Options:
- Detect the failure and fall back to creating a new page
- Detect the failure and return a specific `OutStatus` value (e.g. `STALE_MAPPING`) so the Topic can surface a meaningful message
- Both

**Recommended approach:** use the existing `OutStatus` differentiation framework — if `Update_page_content_Existing_Branch` fails, route to a new `OutStatus` value rather than silently continuing. Keeps the logic clean and consistent with what was built today.

---

## UJ4 — Three gaps

### UJ4a — Section choice (multiple sections with same name)

**The problem:** if `Filter_OneNote_Section_Recurring` returns more than one result (two sections with the same name), `Apply_to_each_Existing_Section` iterates all of them, updating multiple pages. No disambiguation logic exists.

**Note:** `SETUP_SECTION_AMBIGUOUS` is already wired into `OutStatus` (built today) and fires when the section *creation* filter returns >1 result. But this is a different point — the existing-page update path. Worth checking whether the same signal covers both or whether a separate detection is needed.

### UJ4b — Blank `SeriesMasterId` fallback

**The problem:** if `SeriesMasterId` is empty (can happen for some meeting types), `Filter_Existing_Mapping` matches on an empty string, potentially returning wrong rows or all rows.

**Recommended fix:** add a guard in `Filter_Existing_Mapping` or upstream to detect blank `SeriesMasterId` and route to a safe fallback (e.g. treat as one-off, or return `RECURRING_SETUP_REQUIRED`).

### UJ4c — `Topic.SectionRetryCount`

**The problem:** no retry logic if section creation fails transiently. A single `CreateSectionInNotebook` failure kills the whole capture with no recovery path.

**Recommended fix:** a simple retry loop (Do Until) around the section creation, with a counter variable. Needs design thought — how many retries, what delay, what `OutStatus` on exhaustion.

---

## UJ5 — Two gaps

### UJ5a — Reword/retry option

**The problem:** if the agent returns an error, the user has no way to retry without starting the whole flow over from scratch (saying "capture meeting notes" again).

**Recommended fix:** on the `C12_Error` branch in the Topic, offer the user an explicit "Try again" option that loops back to `C10_Call_FlowB_Create_Page` rather than ending the conversation.

### UJ5b — Explicit Stop

**The problem:** no way for the user to cancel mid-flow (e.g. if they triggered a capture by accident or changed their mind after seeing the candidate list).

**Recommended fix:** add a "Cancel" / "Stop" option to the `C6_Ask_SelectedNumber` prompt, handled in the condition group alongside P/N/date/number options.

---

## After UJ3/4/5 are closed

David is observing occasional errors when capturing a second or third meeting from the same recurring series. This is the next investigation thread after UJ3/4/5 are done. Likely related to the mapping row / `Get_items` timing or the `Filter_Pages_By_Title` date-matching — but diagnose from Activity trace evidence first, don't guess.

---

## Recommended order for this evening

1. UJ5a and UJ5b — Topic-only changes, no Flow B canvas edits, lowest corruption risk, good warm-up.
2. UJ4b — small guard expression, low complexity.
3. UJ3 — needs the most design thought, tackle with a fresh head.
4. UJ4a and UJ4c — most complex, leave until UJ3 and UJ4b are done.

---
*Written 22 August 2026, end of afternoon session.*
