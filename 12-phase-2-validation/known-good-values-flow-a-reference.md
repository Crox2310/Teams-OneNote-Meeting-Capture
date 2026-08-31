# Known-good values — Flow A reference (PA - Resolve Meeting Selection - v1 Clean Build)

## Purpose

Companion to `known-good-values-master-reference.md`, which covers Flow B only. This document covers **Flow A**. Created 23 August 2026 after Flow Checker found 2 corrupted actions in Flow A at session start — the first confirmed corruption incident in Flow A (previously only Flow B and Email Triage were affected).

**Last verified against live flow:** 31 August 2026 (Stage 2 — `FA40_Compose_OutCandidateList_Multi` expression updated to prepend date header).

---

## Corruption incident — 23 August 2026 (session start) — RESOLVED

**Symptom:** Flow Checker showed 2 operation errors on opening Flow A:
- `FA33A Set varCandidateListText Empty` — Parameter error: 'Value' is required.
- `FA34A Set varCandidateIndex One` — Parameter error: 'Value' is required.

**Fix applied and confirmed (23 Aug):**
| Action | Correct value | Type |
|---|---|---|
| `FA33A_Set_varCandidateListText_Empty` | `@string('')` | string (expression) |
| `FA34A_Set_varCandidateIndex_One` | `1` | integer literal |

**Status:** ✅ Resolved.

---

## Full flow structure (last verified 31 August 2026)

### Trigger
`Request` (Skills kind), schema fields: `text_1` (InSelectedNumber), `text_3` (DateContext). Both required.

### Top-of-flow InitializeVariable / Compose chain
| Action | Type | Value |
|---|---|---|
| `FA02_Init_varInSelectedNumber` | InitializeVariable (string) | `@if(or(empty(triggerBody()?['text_1']), equals(trim(replace(triggerBody()?['text_1'], '"', '')), '')), '', coalesce(triggerBody()?['text_1'], ''))` |
| `FA04_Init_varDateContext` | InitializeVariable (string) | `@triggerBody()?['text_3']` |
| `FA03A_DEBUG_RawInputs` | Compose | `"UserSearchText": \n  "InSelectedNumber": @{variables('varInSelectedNumber')}\n  "OriginalUserSearchText": \n  "DateContext": @{variables('varDateContext')}\n  "MaxCandidates": \n` |
| `FA06_Compose_StartOfDayUtc` | Compose | `@formatDateTime(if(empty(trim(coalesce(variables('varDateContext'), ''))), utcNow(), variables('varDateContext')), 'yyyy-MM-ddT00:00:00Z')` |
| `FA07_Compose_EndOfDayUtc` | Compose | `@formatDateTime(if(empty(trim(coalesce(variables('varDateContext'), ''))), utcNow(), variables('varDateContext')), 'yyyy-MM-ddT23:59:59Z')` |
| `FA08_Get_calendar_view_of_events` | OpenApiConnection (Office365 `GetEventsCalendarViewV3`) | `calendarId`: (David's calendar GUID), `startDateTimeUtc`: `@outputs('FA06_Compose_StartOfDayUtc')`, `endDateTimeUtc`: `@outputs('FA07_Compose_EndOfDayUtc')` |
| `FA08A_DEBUG_RawConnectorOutput` | Compose | `@body('FA08_Get_calendar_view_of_events')` |
| `FA09_RAW_CandidateArray_DoNotUseDownstream` | Compose | `@body('FA08_Get_calendar_view_of_events')?['value']` |
| `FA09B_Filter_ExcludeLeaveAndPeriodEntries` | Query (Filter array) | Filters out holiday/leave/A-L/OOO/out of office/bank holiday/smarter working/period reminders/manage email & teams/quiet hour patterns |
| `FA09C_Sort_CandidatesByStartTime` | Compose | `@sort(body('FA09B_Filter_ExcludeLeaveAndPeriodEntries'), 'start')` |
| `FA10_Initialize_varCandidates` | InitializeVariable (array) | *(none)* |
| `FA34_Initialize_varCandidateIndex` | InitializeVariable (integer) | *(none)* |
| `FA11_Apply_to_each_Candidates` | Foreach over `@outputs('FA09C_Sort_CandidatesByStartTime')` | Contains `FA12_Append_to_array_varCandidates` + `FA11A_Increment_varCandidateIndex` |
| `FA13_Compose_MatchCount` | Compose | `@length(outputs('FA09C_Sort_CandidatesByStartTime'))` |
| `FA33_Initialize_varCandidateListText` | InitializeVariable (string) | *(none)* |
| `FA14_Compose_CandidateList` | Compose | `@concat(string(variables('varCandidateIndex')), '. ', item()?['subject'], ' (', if(empty(item()?['start']?['dateTime']), 'All day', formatDateTime(convertTimeZone(item()?['start']?['dateTime'], 'UTC', 'GMT Standard Time'), 'HH:mm')), ')')` |
| `FA15_Compose_IsSelectionMode` | Compose | `@and(not(empty(trim(variables('varInSelectedNumber')))), not(contains(createArray('NONE','P','N'), toUpper(trim(variables('varInSelectedNumber'))))))` |

### FA27 — Condition: MatchCount (2 cases)
**If** `@outputs('FA13_Compose_MatchCount')` equals `0`:
- `FA27B_Compose_OutStatus_NoMatch` → `NO_MATCH`
- `FA27C_Compose_OutMatchCount_NoMatch` → `0`
- `FA27D_Compose_OutCandidateList_NoMatch` → `@string('')`
- `FA27E_Compose_OutMeetingTitle_NoMatch` → `@string('')`
- `FA27F_Compose_OutCalendarEventId_NoMatch` → `@string('')`
- `FA27G_Compose_OutIsRecurring_NoMatch` → `@string('')`
- `FA27H_Compose_OutSeriesMasterId_NoMatch` → `@string('')`

**Else → FA27A — Condition: MatchCountIsOne**
**If** `@outputs('FA13_Compose_MatchCount')` equals `1`:
- `FA28_Compose_SingleEvent` → `@outputs('FA09C_Sort_CandidatesByStartTime')[0]`
- `FA28A_Compose_OutIsRecurring` → `@if(empty(coalesce(outputs('FA28_Compose_SingleEvent')?['seriesMasterId'], '')), 'false', 'true')`
- `FA28B_Compose_OutSeriesMasterId` → `@coalesce(outputs('FA28_Compose_SingleEvent')?['seriesMasterId'], '')`
- `FA29_Compose_OutMeetingTitle` → `@coalesce(outputs('FA28_Compose_SingleEvent')?['subject'], '')`
- `FA29B_Compose_OutBodyPreview_Single` → `@substring(coalesce(outputs('FA28_Compose_SingleEvent')?['body'], ''), 0, max(indexOf(coalesce(outputs('FA28_Compose_SingleEvent')?['body'], ''), '</body>'), 0))`
- `FA29C_Compose_OutOnlineMeetingUrl_Single` → `@coalesce(outputs('FA28_Compose_SingleEvent')?['onlineMeeting']?['joinUrl'], '')`
- `FA30_Compose_OutCalendarEventId` → `@coalesce(outputs('FA28_Compose_SingleEvent')?['id'], '')`
- `FA31_Compose_OutMatchCount_Single` → `1`
- `FA32_Compose_OutCandidateList_Single` → `@string('')`

**Else (multi-match branch):**
- `FA33A_Set_varCandidateListText_Empty` (SetVariable) → `varCandidateListText` = `@string('')` ✅ *corrected 23 Aug*
- `FA34A_Set_varCandidateIndex_One` (SetVariable) → `varCandidateIndex` = `1` ✅ *corrected 23 Aug*
- `FA35_Apply_to_each_CandidateArray_ForList` (Foreach over `@outputs('FA09C_Sort_CandidatesByStartTime')`):
  - `FA36_Append_to_string_varCandidateListText` → `@concat(string(variables('varCandidateIndex')), '. ', coalesce(item()?['subject'], 'Untitled meeting'), decodeUriComponent('%0D%0A'))`
  - `FA37_Increment_varCandidateIndex` → increment `varCandidateIndex` by `1`
- `FA38_Compose_OutStatus_Multi` → `MULTIPLE_MATCHES`
- `FA39_Compose_OutMatchCount_Multi` → `@string(outputs('FA13_Compose_MatchCount'))`
- `FA40_Compose_OutCandidateList_Multi` → `@concat('Meetings for ', formatDateTime(variables('varDateContext'), 'ddd d MMM yyyy'), decodeUriComponent('%0D%0A'), variables('varCandidateListText'))` ✅ *updated 31 Aug — prepends date header*
- `FA41_Compose_OutMeetingTitle_Multi` → `@string('')`
- `FA42_Compose_OutCalendarEventId_Multi` → `@string('')`
- `FA43A_Compose_OutIsRecurring_Multi` → `@string('')`
- `FA43B_Compose_OutSeriesMasterId_Multi` → `@string('')`

### FA17 — Condition: IsSelectionMode (2 cases)
**If** `@outputs('FA15_Compose_IsSelectionMode')` equals `true`:
- `FA16_Compose_SelectedIndex` → `@if(or(equals(trim(variables('varInSelectedNumber')), ''), not(equals(string(mul(int(if(equals(trim(variables('varInSelectedNumber')), ''), '1', trim(variables('varInSelectedNumber')))), 1)), trim(variables('varInSelectedNumber'))))), 0, sub(int(trim(variables('varInSelectedNumber'))), 1))`
- **FA18 — Condition: SelectedIndexInRange:**
  - **If** in range:
    - `FA19_Compose_SelectedEvent` → `@outputs('FA09C_Sort_CandidatesByStartTime')[outputs('FA16_Compose_SelectedIndex')]`
    - `FA20_Compose_OutMeetingTitle` → `@coalesce(outputs('FA19_Compose_SelectedEvent')?['subject'], '')`
    - `FA21_Compose_OutCalendarEventId` → `@coalesce(outputs('FA19_Compose_SelectedEvent')?['id'], '')`
    - `FA20B_Compose_OutBodyPreview` → `@if(equals(coalesce(outputs('FA19_Compose_SelectedEvent')?['body'], ''), ''), '', substring(coalesce(outputs('FA19_Compose_SelectedEvent')?['body'], ''), 0, max(indexOf(coalesce(outputs('FA19_Compose_SelectedEvent')?['body'], ''), '</body>'), 0)))`
    - `FA20C_Compose_OutOnlineMeetingUrl` → `@coalesce(outputs('FA19_Compose_SelectedEvent')?['onlineMeeting']?['joinUrl'], '')`
    - `FA19B_Compose_OutIsRecurring_Resolved` → `@if(empty(coalesce(outputs('FA19_Compose_SelectedEvent')?['seriesMasterId'], '')), 'false', 'true')`
    - `FA19C_Compose_OutSeriesMasterId_Resolved` → `@coalesce(outputs('FA19_Compose_SelectedEvent')?['seriesMasterId'], '')`
    - `FA22_Compose_OutMatchCount_Resolved` → `1`
    - `FA23_Compose_OutCandidateList_Resolved` → `@string('')`
  - **Else (out of range):**
    - `FA24_Compose_OutStatus_InvalidSelection` → `ERROR`
    - `FA25_Compose_OutMatchCount_Error` → `0`
    - `FA26_Compose_OutCandidateList_Error` → `Selected number is out of range.`

### Response action
`FA43_Respond_to_agent` — Response (Skills kind), statusCode 200. Body fields (`status`, `matchcount`, `candidatelist`, `meetingtitle`, `calendareventid`, `isrecurring`, `seriesmasterid`, `onlinemeetingurl`, `bodypreview`) each `coalesce()` across the three branch outcomes with a final literal fallback.

---

## How to use during a corruption incident

1. Confirm which actions lost their value (Peek Code + Flow Checker).
2. Cross-check this table. Check "Last verified" date against the most recent session note for any subsequent changes.
3. Paste back exactly — do not retype from memory. Note: some empty-string values use `@string('')` expression form rather than a bare literal — check the table above for the exact form per action.
4. Save draft, run Flow Checker, then Publish before testing.
5. Update this doc's "Last verified" date if anything needed correcting.

---
*Created 23 August 2026. Updated 31 August 2026 with FA40 date-header expression (Stage 2) and FA09B/FA09C additions (FR-02/FR-01, missed from original capture). Companion to `known-good-values-master-reference.md` (Flow B).*
