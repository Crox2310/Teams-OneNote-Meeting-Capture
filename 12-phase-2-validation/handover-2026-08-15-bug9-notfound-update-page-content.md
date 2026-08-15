# Bug 9 — NotFound on Update page content (Existing Branch) — discovered during Bug 5 retest, 15 August 2026

## Status: NEW, UNFIXED, under investigation same session

## Context

While retesting Bug 5 (one-off recapture, previously failing with empty `sectionId` on `Create_Page_OneOff`), a second run using the same `MeetingId` as an earlier successful one-off capture today revealed that **Bug 5's original symptom is resolved**, but exposed a new, distinct failure.

## Bug 5 status: partially confirmed fixed

- Test 1 (fresh `MeetingId: bug5-retest-15aug-2`, novel title, `IsRecurring: false`): routed correctly through `Create_OneNote_Page` (standard path, since no existing page was found) — succeeded in 0.7s. `Create_Page_OneOff` correctly skipped (dependent condition not met).
- Test 2 (same `MeetingId` reused, forcing an existing-mapping match): `Condition Is Genuine Existing Page` correctly evaluated **True** — `varOneNoteResolverResult` correctly resolved to `ExistingMapping`/`ExistingSection` this time. This means the flow correctly routed to the "update existing page" branch rather than falling through to `Create_Page_OneOff` with an empty `sectionId` — **the original Bug 5 symptom did not reproduce.**

This is a real, positive outcome — Bug 5's root cause (corrupted `varTargetSectionPagesUrl`/`varOneNoteResolverResult` SetVariable actions, fixed earlier this session) appears to have resolved the empty-`sectionId` failure as expected.

## New finding: Bug 9 — `NotFound` on `Update page content Existing Branch`

In Test 2, having correctly routed to the "update existing page" branch, the flow failed at:

- **Action**: `Update page content Existing Branch` (inside `Apply to each Existing Section`, inside `Condition Is Genuine Existing Page` → True branch)
- **Error**: `NotFound`
- **Timestamp**: 15 August 2026, 17:47

### Working theory

`Update page content Existing Branch`'s `pageId` parameter is `@outputs('Compose_ExistingPageId')`, which is derived from `@last(split(variables('varOutputPageSelfUrl'), '/'))`. `varOutputPageSelfUrl` is set earlier in the run from the SharePoint mapping row's `PageSelfUrl` field.

Test 1 (the run that created the mapping row) created the page via the **standard** `Create_OneNote_Page` path, not `Create_Page_OneOff`. It's possible the write-back to SharePoint's `PageSelfUrl`/`PageWebUrl` columns on that path (`OF09a — Send an HTTP request to SharePoint (OneOff)`, or the equivalent on the standard path) did not correctly persist the actual created page's self-URL — meaning Test 2 read a stale, empty, or malformed `PageSelfUrl` from the mapping row, producing a `pageId` that doesn't correspond to any real page in OneNote, hence `NotFound`.

### Not yet confirmed

- Whether the SharePoint mapping row from Test 1 actually has a valid `PageSelfUrl` value (needs checking directly in the `RecurringMeetingSectionMap` list — same list already viewed earlier this session for the Bug 8 test).
- Whether this is a corruption artifact or a genuine, previously-unexposed logic gap in the OneOff mapping-write step, unrelated to the corruption cluster.
- Whether this reproduces on the **recurring** branch's equivalent update-existing-page logic, or is isolated to the one-off path.

## Next steps

1. Check the `RecurringMeetingSectionMap` SharePoint list directly for the mapping row created during Test 1 (MeetingId `bug5-retest-15aug-2`) and inspect its `PageSelfUrl` / `PageWebUrl` columns for validity.
2. Trace `OF09a — Send an HTTP request to SharePoint (OneOff)`'s body payload (Peek Code / run history) to confirm what value it actually wrote for `PageSelfUrl` on Test 1's run.
3. Compare against the working recurring-branch equivalent (`Send_an_HTTP_request_to_SharePoint`, non-OneOff) to check for a parity gap between the two branches' mapping-write logic.

---

*Logged 15 August 2026, same session as the 26-action corruption recovery and Bug 8 confirmation. See `handover-2026-08-15-session2-part2-recovery-complete-published.md` for full session context.*
