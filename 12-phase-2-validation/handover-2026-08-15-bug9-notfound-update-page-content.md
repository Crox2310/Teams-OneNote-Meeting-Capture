# Bug 9 — NotFound on Update page content (Existing Branch) — discovered during Bug 5 retest, 15 August 2026

## Status: NEW, UNFIXED, root cause narrowed (updated same session)

## Context

While retesting Bug 5 (one-off recapture, previously failing with empty `sectionId` on `Create_Page_OneOff`), a second run using the same `MeetingId` as an earlier successful one-off capture today revealed that **Bug 5's original symptom is resolved**, but exposed a new, distinct failure.

## Bug 5 status: confirmed fixed (original symptom)

- Test 1 (fresh `MeetingId: bug5-retest-15aug-2`, novel title, `IsRecurring: false`): routed correctly through `Create_OneNote_Page` (standard path, since no existing page was found) — succeeded in 0.7s. `Create_Page_OneOff` correctly skipped (dependent condition not met).
- Test 2 (same `MeetingId` reused, forcing an existing-mapping match): `Condition Is Genuine Existing Page` correctly evaluated **True** — `varOneNoteResolverResult` correctly resolved to `ExistingSection`. The flow correctly routed to the "update existing page" branch rather than falling through to `Create_Page_OneOff` with an empty `sectionId` — **the original Bug 5 symptom did not reproduce.**

This is a real, positive outcome — Bug 5's root cause (corrupted `varTargetSectionPagesUrl`/`varOneNoteResolverResult` SetVariable actions, fixed earlier this session) appears to have resolved the empty-`sectionId` failure as expected.

## Bug 9 finding: `NotFound` on `Update page content Existing Branch`

- **Action**: `Update page content Existing Branch` (inside `Apply to each Existing Section`, inside `Condition Is Genuine Existing Page` → True branch)
- **Error**: `NotFound`
- **Timestamp**: 15 August 2026, ~17:47

### Root cause narrowed — CONFIRMED, not just theory

Checked the `RecurringMeetingSectionMap` SharePoint list directly after Test 2. **Result: the list still contains only one row — the original "SC Eng Leadership Weekly" recurring mapping. There is NO row for `MeetingId: bug5-retest-15aug-2` or `MeetingTitle: Bug 5 Retest 15 Aug`.** All mapping-specific columns (`PageSelfUrl`, `PageWebUrl`, `MeetingId`, `SectionId`) are empty across the board in the only existing row (which is the recurring-branch row and was never expected to have MeetingId populated — recurring uses `SeriesMasterId` instead).

**This means: Test 1's page creation succeeded in OneNote, but the corresponding SharePoint mapping write for the one-off branch (`OF09a — Send an HTTP request to SharePoint (OneOff)`) either did not run, or ran but did not persist a row.**

This reframes the bug:
- Test 2's `OF01 — Filter_Existing_Mapping_OneOff` (which queries the SharePoint list by `MeetingId`) would have found **zero rows** for `bug5-retest-15aug-2` — meaning the SharePoint-based lookup should have produced `PAGE_NOT_FOUND`.
- But the run instead correctly identified the section already existed via a **different signal** — OneNote's own section listing (`Get_Sections_OneOff` / `Filter_OneNote_Section_OneOff`) found the section by name, independent of the SharePoint mapping table, and set `varOneNoteResolverResult = ExistingSection` via that path.
- The flow then tried to update the "existing page" using a page reference (`varOutputPageSelfUrl` / `Compose_ExistingPageId`) that was never legitimately populated, because there was no SharePoint mapping row to source it from — producing the `NotFound` error when it tried to update a page ID that doesn't actually resolve.

**Conclusion: this is most likely a genuine logic gap — `OF09a` failing to write or persist a mapping row on the one-off Created path — rather than a corruption/data-integrity issue.** Worth checking whether `OF09a` even executed during Test 1's run (via Activity/run history for that earlier run) to confirm whether it silently failed, was skipped, or ran but the write itself didn't take.

## Next steps

1. **Pull Test 1's run history** (the run that created the page, ~17:42) and check whether `OF09a — Send an HTTP request to SharePoint (OneOff)` executed, and if so, inspect its raw outputs — did SharePoint actually receive and process the insert?
2. If `OF09a` did run successfully, check whether the issue is instead that `OF09b-i — Condition Should Insert Mapping (OneOff)`'s gating condition (`length(body('OF01...')) == 0`) evaluated incorrectly, skipping the insert even though it should have run.
3. Compare against the working recurring-branch equivalent (`Send_an_HTTP_request_to_SharePoint`, non-OneOff, which reliably writes the "SC Eng Leadership Weekly" row) to check for a parity gap between the two branches' mapping-write logic.
4. Consider: is `Update page content Existing Branch`'s reliance on `varOutputPageSelfUrl` (sourced from the SharePoint mapping) too fragile for a scenario where the OneNote section is found directly but no mapping row exists? A more robust fix might source the page reference from the OneNote section listing itself in this fallback case, rather than assuming the SharePoint mapping is always populated.

---

*Logged 15 August 2026, updated same session with confirmed root-cause narrowing. See `handover-2026-08-15-session2-part2-recovery-complete-published.md` for full session context.*
