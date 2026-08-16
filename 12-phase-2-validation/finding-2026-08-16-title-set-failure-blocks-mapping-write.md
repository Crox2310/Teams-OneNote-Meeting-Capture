# Finding — `Set_PageTitle_Recurring` failure blocks SharePoint mapping write entirely, causing guaranteed duplicate pages on every retry

**Found:** 16 August 2026, live at work, following a `404`/OneNote-error-20102 failure on `Set_PageTitle_Recurring` for "NH Performance Feedback - David, Simon & Jin Connect" (recurring branch).
**Status:** root cause confirmed. This upgrades the severity of the previously-logged reliability issue in `handover-2026-08-16-page-title-fix-recurring-confirmed.md` — the consequence is worse than originally understood.

---

## What was found

After `Set_PageTitle_Recurring` failed with the known intermittent `404` (see prior handover for the underlying propagation-race cause), OneNote showed **two separate pages**, both titled `"18 Aug 2026 - NH Performance Feedback - David, Simon & Jin Connect"`, both with identical content — created by two separate capture attempts for what should be the same meeting occurrence.

## Root cause — action ordering

`Set_PageTitle_Recurring` runs **before** `OF09-Gate` in the flow, and `OF09-Gate` is what performs the actual SharePoint mapping write (`HTTP_Update_SP_PageSelfUrl`, `Set_varPageAction_Created`, `Set_varOutputPageSelfUrl_Created`, `Set_varOutputPageLink_Created`) for a newly-created page.

When `Set_PageTitle_Recurring` fails, **the flow run terminates before ever reaching `OF09-Gate`.** This means:

1. `Create_OneNote_Page` has already succeeded — a real page genuinely exists.
2. But the SharePoint `RecurringMeetingSectionMap` row that should record "this meeting now has a page" **never gets written**, because the write step is downstream of the point of failure.
3. On the **next** capture attempt for the same meeting, SharePoint still shows no mapping — so the flow correctly (by its own logic) treats it as a brand-new meeting and creates **another** new page.
4. This isn't a duplicate-detection bug — it's the direct, logical, unavoidable consequence of the mapping-write step never being reached. **Every retry while this reliability issue is live will produce another duplicate page, not a recovery.**

## Why this raises the priority of the underlying fix

The `Set_PageTitle_Recurring` race condition was previously logged (16 August, page-title-fix handover) as an intermittent reliability annoyance — a title occasionally not getting set. **This finding shows the actual consequence is more serious**: it can silently and permanently prevent a meeting from ever being correctly tracked, producing an unbounded number of duplicate pages with each retry, until the underlying race condition happens not to occur.

## Immediate action taken

Both duplicate pages for this meeting were identified; no further retries were attempted tonight, since doing so would only create additional duplicates rather than resolve anything while the underlying issue remains unfixed.

## Recommended fix priority

**This should be treated as a higher-priority fix than originally scoped.** The previously-recommended approach (replace the abandoned Delay-based mitigation with a `Do until` retry/poll pattern around the page-ID confirmation, avoiding the Express-mode incompatibility that blocked the Delay approach) remains the right direction — but should now be prioritised above the one-off branch's stale-mapping edge case and the tail-section anomaly, given this failure mode actively creates duplicate, user-visible artifacts on ordinary recurring-meeting use, not just an edge case.

**Secondary, independent improvement worth considering**: reordering the flow so the SharePoint mapping write happens *before* (or independently of) the title-set step, so that even if title-setting fails, the meeting is still correctly recorded as having a page — preventing duplicate creation regardless of whether the title race condition itself is ever fully eliminated. This would be a more robust fix than reliability-improving the race condition alone, since it removes the single point of failure that currently blocks mapping persistence.

---

**Status: root cause fully understood. Both duplicate pages identified, left in place pending next session's cleanup (no further retries attempted). Fix priority raised — this is no longer a minor reliability annoyance but a real, repeatable cause of duplicate user-facing content on the recurring capture path.**
