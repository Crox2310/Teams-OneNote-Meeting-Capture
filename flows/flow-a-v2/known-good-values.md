# Flow A v2 — Known-good values

**Flow name:** PA - Resolve Meeting Selection - v2
**Created:** 26 September 2026
**Last confirmed green:** 26 September 2026
**Build status:** Complete — published and tested

## Platform constraints confirmed during build

| Constraint | Detail |
|---|---|
| `indexOf()` array support | NOT supported — string only. varCandidateIndex counter used instead. |
| `InitializeVariable` nesting | Cannot be inside Scope or Condition — must be top-level |
| Dynamic Content picker | Inserts internal IDs not action names — always type expressions manually |
| Trigger key assignment | New Designer assigns keys sequentially: `text`, `text_1`, `text_2`... |

## Top-level variables (outside Scope_FlowA)

| Ref | Action name | Value | Type |
|---|---|---|---|
| TV01 | TV01 Initialise Candidate List Text | (blank) | string |
| TV02 | TV02 Initialise Candidate Index | `1` | integer |

⚠️ TV02 value wipes on Designer open (same corruption pattern as v1 FA34A). Check at session start. Restore value to `1` if blank.

## Scope_CalendarFetch

| Ref | Action name | Expression |
|---|---|---|
| CA01 | CA01 Compose DateContext | `@coalesce(triggerBody()?['text_1'], utcNow())` |
| CA02 | CA02 Compose StartOfDay | `@formatDateTime(if(empty(trim(coalesce(outputs('CA01_Compose_DateContext'), ''))), utcNow(), outputs('CA01_Compose_DateContext')), 'yyyy-MM-ddT00:00:00Z')` |
| CA03 | CA03 Compose EndOfDay | `@formatDateTime(if(empty(trim(coalesce(outputs('CA01_Compose_DateContext'), ''))), utcNow(), outputs('CA01_Compose_DateContext')), 'yyyy-MM-ddT23:59:59Z')` |
| CA04 | CA04 Get Calendar Events | calendarId: `AAMkAGY0OGU4Mzk5LWQ4NTYtNDU4MS1hY2YyLTQxOWYwZjhiMWM1ZQBGAAAAAADWkXK1vW2mQ4SwNGpyD7SzBwB8mPnOPkRmT5-MxNoNopoPAAAAAAEGAAB8mPnOPkRmT5-MxNoNopoPAACPZPifAAA=` |
| CA05 | CA05 Compose Raw Candidates | `@body('CA04_Get_Calendar_Events')?['value']` |
| CA06 | CA06 Filter Exclude Leave And Periods | `@not(or(contains...))` — see scope-flow-a.md |
| CA07 | CA07 Compose Sorted Candidates | `@sort(body('CA06_Filter_Exclude_Leave_And_Periods'), 'start')` |

## Scope_CandidateResolution

| Ref | Action name | Expression |
|---|---|---|
| CR01 | CR01 Compose Match Count | `@length(outputs('CA07_Compose_Sorted_Candidates'))` |
| CR02 | CR02 Compose Display Date | `@formatDateTime(concat(outputs('CA01_Compose_DateContext'), 'T12:00:00'), 'ddd d MMM yyyy')` |

## Scope_NoMatch

NM01 condition: `outputs('CR01_Compose_Match_Count')` equals `0`

All NM02–NM11 are `@string('')` except NM02 (`NO_MATCH`) and NM03 (`@string(0)`)

## Scope_SingleMatch

SM01 condition: `outputs('CR01_Compose_Match_Count')` equals `1`

Key values: SM14 = `MULTIPLE_MATCHES` (intentional), SM15 = `@string(1)`, SM16 = `@string('')`

## Scope_MultiMatch

MM01 condition: `outputs('CR01_Compose_Match_Count')` greater than `1`

MM02 filter: `OccurrenceDate eq '@{formatDateTime(outputs('CA01_Compose_DateContext'), 'yyyy-MM-dd')}'`

Loop: MM04a (Filter Array OR match) → MM04b (status label) → MM04c (AppendToStringVariable) → MM04d (IncrementVariable by 1)

MM04b expression: `@if(equals(coalesce(first(body('MM04a_Filter_Mapping_Match'))?['RecapCaptured'], false), true), ' - **Mtg Notes**', if(greater(length(body('MM04a_Filter_Mapping_Match')), 0), ' - **Captured**', ''))`

MM04c value: `@concat(string(variables('varCandidateIndex')), '. ', coalesce(items('MM04_Build_Candidate_List_Loop')?['subject'], 'Untitled meeting'), outputs('MM04b_Compose_Status_Label'), decodeUriComponent('%0D%0A'))`

## Scope_Response

RP01 uses `coalesce()` across all three branch outputs with fallback. See scope-flow-a.md for full expressions.

## Open backlog items identified during build

| Item | Detail |
|---|---|
| Filter gap | `A/L` (with slash or space) and `Day Off` not caught by current filter. `a-l` term only matches hyphenated form. |
| Single-match test | Deferred — calendar too full to find single-meeting day easily. Test naturally at Topic switchover. |
| No-match test | Deferred — calendar too full. No-match path is simple and low-risk. |
| Topic switchover | C2 call must be updated: `text` = InSelectedNumber, `text_1` = DateContext (v2 keys differ from v1). |
