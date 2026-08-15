# Bug 9 — final root cause, fixes applied, and a new corruption variant observed — 15 August 2026

## Status: TWO REAL FIXES APPLIED. Re-testing in progress. One new corruption anomaly logged, unresolved.

## Summary of the full investigation arc

Bug 9 (`NotFound` on `Update page content Existing Branch`) went through several incorrect and partial theories before reaching the true root cause. This document supersedes the two earlier Bug 9 files' conclusions and should be treated as the current state of understanding.

## Confirmed real root cause #1: wrong SharePoint column read

`Compose_ExistingPageSelfUrl` (recurring branch) and `OF02 — Compose_ExistingPageSelfUrl_OneOff` (one-off branch) were both reading the SharePoint `PageWebUrl` column instead of `PageSelfUrl`, despite being named for "self URL." `PageWebUrl` holds a OneNote client deep-link (`https://.../Meeting%20Notes?wd=target%28...`), which is not a valid `pageId` for the `UpdatePageContent` OneNote API call — hence `NotFound`.

**Confirmed via direct evidence**: `Create_OneNote_Page`'s raw output for a real run showed `self` = a correct API page reference, while `links.oneNoteWebUrl.href` = the deep-link value that was actually ending up downstream. This proved the correct value exists in the OneNote API response, but the flow was reading the wrong field.

**Fix applied**: both `Compose_ExistingPageSelfUrl` and `OF02 — Compose_ExistingPageSelfUrl_OneOff` had their expressions changed from `?['PageWebUrl']` to `?['PageSelfUrl']`. Flow Checker confirmed clean after each isolated save.

## Confirmed real root cause #2: OF05a missing its value field, then found intact again

After fix #1 was applied, the Bug 9 test was re-run and **still failed identically** — `Update page content Existing Branch` still received a deep-link-style `pageId`, unchanged from before the fix.

Traced downstream: `OF05a — Set varFinalExistingPageSelfUrl (OneOff)` is the action that takes `OF02`'s (now-corrected) output and writes it into `varFinalExistingPageSelfUrl`, which then flows into `varOutputPageSelfUrl` → `Compose_ExistingPageId` → the `pageId` used by `Update page content Existing Branch`.

**When first checked, `OF05a`'s code showed NO `value` field at all** — only `name`, missing `value` entirely:
```json
{
  "type": "SetVariable",
  "inputs": {
    "name": "varFinalExistingPageSelfUrl"
  },
  "runAfter": { "OF04_—_Compose_Match_Count_OneOff": ["SUCCEEDED"] }
}
```
This fully explains the continued failure: even with `OF02` corrected, `OF05a` was never actually writing that corrected value into the variable that downstream logic depends on.

**Before any edit was made to fix this**, the action was checked again and **the value field was found to be present and correct**:
```json
{
  "type": "SetVariable",
  "inputs": {
    "name": "varFinalExistingPageSelfUrl",
    "value": "@outputs('OF02_—_Compose_ExistingPageSelfUrl_OneOff')"
  },
  "runAfter": { "OF04_—_Compose_Match_Count_OneOff": ["SUCCEEDED"] }
}
```

**This is a new and notable corruption variant, distinct from every other incident logged today.** No edit was made by David between the two checks. The value field went from completely absent to fully correct without any deliberate user action. Possible explanations, none confirmed:
- A background save or publish event self-corrected the field.
- Navigating away from and back to the action triggered a re-read that resolved a display/caching issue rather than reflecting the true stored state at the time of the first check.
- A genuine platform-side auto-heal or delayed-write-commit behavior, consistent with the "blank values take time to settle" mechanism hypothesis logged earlier this session — except here the resolution was toward the *correct* value rather than toward further corruption.

**This should be added to the Microsoft ticket as an additional, distinct data point**: not just values going blank, but a value appearing genuinely absent on one read and correct on a subsequent read with no user edit in between, in the same session, on the same unpublished-then-published flow.

## Current state

- Fix #1 (PageWebUrl → PageSelfUrl on both branches' Compose actions): applied, confirmed via Flow Checker.
- Fix #2 (OF05a's value field): found already correct on the check immediately following the failed re-test — no edit needed or made.
- **Re-testing in progress** to confirm whether the pipeline now works end-to-end with both fixes genuinely in place.

## Next steps

1. Complete the current re-test (fresh one-off capture, then reuse the same MeetingId) and confirm `Update page content Existing Branch` succeeds.
2. If it still fails, check `Compose_ExistingPageId`'s own expression directly, and `Set_varOutputPageSelfUrl_Existing`'s expression, to rule out a third link in the chain being wrong.
3. Add this session's full Bug 9 saga (both real fixes, the false leads along the way, and the OF05a self-resolving anomaly) to the Microsoft support ticket as supplementary evidence of the corruption pattern's behavior.

---

*Logged 15 August 2026, supersedes earlier conclusions in this file's history. See `handover-2026-08-15-session2-part2-recovery-complete-published.md` and `MICROSOFT-SUPPORT-TICKET-DRAFT-2026-08-15.md` for full session context.*
