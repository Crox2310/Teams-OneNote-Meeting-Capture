# Stage 1 session — Safety net build and gate

**Date:** 31 August 2026
**Session type:** Build and gate — Stage 1 of the 29 August backlog.
**Flows changed:** Flow B (`ed112c88-b94b-f111-bec6-002248a38052`) only.
**Topic changed:** No.
**SharePoint list changed:** No.

---

## What was built

### Pre-build: scratch test

Before touching Flow B, confirmed that WDL's `if()` function short-circuits — when the true branch condition is met, the false branch is never evaluated. Tested with `@if(true, 'branch 1 taken', formatDateTime(null, 'd MMM yyyy'))` in `PA - Scratch Diagnostics`. Output was `branch 1 taken`; no error. This confirmed Fix 1's `where` clause is safe.

### Fix 1 — Null guard on `S1_Filter_Pages_By_Title_PreCreate`

The original `where` clause called `formatDateTime(triggerBody()?['text_5'], 'd MMM yyyy')` directly. `text_5` (OccurrenceDate) is not in the trigger's `required` list, so this would throw on any capture without a date. Fixed by wrapping in `if(empty(coalesce(...)))` using `Compose_SafePageTitle` as the fallback for the no-date case:

```
@if(
  empty(coalesce(triggerBody()?['text_5'], '')),
  equals(coalesce(item()?['title'], ''), outputs('Compose_SafePageTitle')),
  contains(coalesce(item()?['title'], ''), formatDateTime(triggerBody()?['text_5'], 'd MMM yyyy'))
)
```

Note: the same unguarded call exists in `Filter_Pages_By_Title` inside `Apply_to_each_Existing_Section` — deferred, lower risk path.

### Write-back chain (S1W01–S1W05)

Five new actions added inside the True branch of `S1_Condition_Title_Safety_Check`, after `S1_Set_varOutputPageLink`. Purpose: when the S1 safety net appends to an existing page instead of creating a new one, heal the missing or deleted mapping row so future captures don't loop through S1 again.

**New variable on main trunk:**
- `varOneOffMappingId` (string, blank) — initialised after `varPageAction` in the main InitializeVariable chain. Needed because `S1_HTTP_Update_SP_Mapping_OneOff` cannot reference `body('S1_Create_Mapping_Item_OneOff')` directly across parallel condition branches.

**S1W01 — `S1_Condition_Is_Recurring_Writeback`**
Condition: `@equals(toLower(string(triggerBody()?['text'])), 'true')` — routes recurring vs one-off write-back.

**S1W02 — `S1_HTTP_Update_SP_Mapping_Recurring`** (True branch)
Send an HTTP request to SharePoint — MERGE on the existing or newly created recurring mapping row. Updates `PageSelfUrl` and `PageWebUrl` from the variables set upstream in the S1 true branch.

Uri: `_api/web/lists/GetByTitle('RecurringMeetingSectionMap')/items(@{if(greater(length(body('Filter_Existing_Mapping')),0), first(body('Filter_Existing_Mapping'))?['ID'], body('Create_Mapping_Item_Recurring')?['ID'])})`

**S1W03 — `S1_Condition_Should_Insert_Mapping_OneOff`** (False branch of S1W01)
Condition: `@length(body('OF01_—_Filter_Existing_Mapping_OneOff'))` equals `0`. If the row is missing, create it (True branch). If it exists, skip creation (False branch, empty).

**S1W04 — `S1_Create_Mapping_Item_OneOff`** (True branch of S1W03)
SharePoint Create item — new mapping row for one-off meetings where the row was deleted. Fields: Title=Mapping, MeetingTitle, SectionPagesUrl, Status=Active, MeetingId. PageSelfUrl and PageWebUrl left blank — written by S1W05 via MERGE.

**`S1_Set_varOneOffMappingId`** (after S1W04, True branch of S1W03)
Stores `@string(body('S1_Create_Mapping_Item_OneOff')?['ID'])` into `varOneOffMappingId` so S1W05 can reference it across the branch boundary.

**S1W05 — `S1_HTTP_Update_SP_Mapping_OneOff`** (after S1W03, False branch of S1W01)
Send an HTTP request to SharePoint — MERGE on the existing or newly created one-off mapping row. Updates `PageSelfUrl` and `PageWebUrl`.

Uri: `_api/web/lists/GetByTitle('RecurringMeetingSectionMap')/items(@{if(greater(length(body('OF01_—_Filter_Existing_Mapping_OneOff')),0), first(body('OF01_—_Filter_Existing_Mapping_OneOff'))?['ID'], variables('varOneOffMappingId'))})`

---

## Gate — Stage 1 safety net

**Scenario:** One-off meeting with existing OneNote page. Mapping row deleted from SharePoint. Meeting recaptured via agent.

**Expected:** S1 branch runs, page appended (not duplicated), mapping row recreated with all URL fields populated.

**Result: PASS.**

Activity trace confirmed:
- `S1_Condition_Title_Safety_Check` → True ✓
- `S1_Update_Page_Content_Append` → 204 ✓
- `S1_Condition_Is_Recurring_Writeback` → False (correct — one-off meeting) ✓
- `S1_Condition_Should_Insert_Mapping_OneOff` → True (row was deleted, new row created) ✓
- `S1_HTTP_Update_SP_Mapping_OneOff` → 204 ✓

OneNote: page appended with correct update note ("A page with a matching title already existed in OneNote with no corresponding mapping row. The automation appended below rather than creating a duplicate page.") — no duplicate ✓

SharePoint: new mapping row for "Mtg - Test One Off meeting" with `SectionPagesUrl`, `PageSelfUrl`, `PageWebUrl`, and `MeetingId` all populated ✓

---

## UJ1–UJ5 regression pass

All five user journeys run after publish and all passed:

| Journey | Result |
|---|---|
| UJ1 — one-off, first capture | ✅ Pass |
| UJ2 — one-off, multi-match | ✅ Pass |
| UJ3 — one-off, recapture | ✅ Pass |
| UJ4 — recurring, first capture | ✅ Pass |
| UJ5 — recurring, recapture | ✅ Pass |

Note on UJ2: a `FlowActionException` appeared in Teams after a successful capture — this is the known Topic checker false-positive pattern. Both Flow A and Flow B activity traces were green. Not a regression.

---

## Decisions made this session

- `varOneOffMappingId` added as a new string variable rather than reusing an existing one — reuse would make debugging harder and is the kind of silent confusing choice that causes bugs.
- S1W05 placed after S1W03 (not inside its False branch) so the MERGE runs regardless of whether a new row was created or one already existed.
- Fix 1 applied to `S1_Filter_Pages_By_Title_PreCreate` only. The same unguarded `formatDateTime` call in `Filter_Pages_By_Title` inside `Apply_to_each_Existing_Section` was noted but deferred — lower risk path, separate fix.

---

## Next action

Stage 2 — Date in the opening prompt. See `design-2026-08-29-target-state-and-backlog.md` for the spec.

Before starting Stage 2, consider:
- Update `amendment-log.md` with today's changes (still has the 23 Aug sprint outstanding too).
- Refresh `known-good-values-master-reference.md` with the new S1W01–S1W05 actions and `varOneOffMappingId`.
- Submit the Microsoft support ticket (`microsoft-discussion-brief-corruption-bug.md`).

---
*Session closed 31 August 2026. Flow B published and all five user journeys validated.*
