# Flow A v2 — Build Instructions

**Flow name:** `PA - Resolve Meeting Selection - v2`
**Description:** Resolves available meetings for a given date, returning a candidate list with capture status indicators for multi-match days, or direct meeting details for single-match days. Rebuilt v2 — Compose-based state passing, Scope-organised, no dead code.
**Status:** Not yet started
**Last updated:** 26 September 2026

---

## Context

This is a full rebuild of Flow A (`PA - Meeting Capture - A - Resolve Meeting Select...`). The existing flow remains in production untouched. This v2 flow is built alongside it and the Topic is only switched over once v2 has passed all three test cases below.

**Key improvements over v1:**
- No InitializeVariable at top of flow — DateContext read directly from trigger via Compose
- No varCandidateIndex SetVariable — replaced with inline `indexOf()` expression
- Dead code removed — FA15–FA26 (IsSelectionMode branch, always `'NONE'`) not rebuilt
- All state passed via `outputs('ActionName')` references — not variables
- Scope-organised — one outer Scope, six inner Scopes
- Known-good values maintained in GitHub as each Scope is confirmed green

---

## Trigger contract

See `trigger-contract.md` in this folder. Summary:
- `text_1` (InSelectedNumber, string, required) — always `'NONE'` from Topic
- `text_3` (DateContext, string, required) — ISO date string for the day to browse

## Output contract

See `output-contract.md` in this folder. Summary:
- 10 fields: status, matchcount, candidatelist, meetingtitle, calendareventid, isrecurring, seriesmasterid, onlinemeetingurl, bodypreview, endtime
- All coalesced across branch Composes in RP01 Respond To Agent

---

## Scope structure

```
Trigger
└── Scope_FlowA (outer — full flow Peek Code)
    ├── Scope_CalendarFetch
    ├── Scope_CandidateResolution
    ├── Scope_NoMatch
    ├── Scope_SingleMatch
    ├── Scope_MultiMatch
    └── Scope_Response
```

---

## Build progress tracker

| Scope | Status | Peek Code pushed | Known-good values updated |
|---|---|---|---|
| Trigger setup | ⏳ Not started | — | — |
| Scope_FlowA (outer) | ⏳ Not started | — | — |
| Scope_CalendarFetch | ⏳ Not started | — | — |
| Scope_CandidateResolution | ⏳ Not started | — | — |
| Scope_NoMatch | ⏳ Not started | — | — |
| Scope_SingleMatch | ⏳ Not started | — | — |
| Scope_MultiMatch | ⏳ Not started | — | — |
| Scope_Response | ⏳ Not started | — | — |
| Testing (3 test cases) | ⏳ Not started | — | — |
| Topic switchover | ⏳ Not started | — | — |

---

## Before you start

- Do NOT delete or modify the existing Flow A. Build entirely alongside it.
- Have PA - Scratch Diagnostics open in a separate tab.
- Have GitHub open at `flows/flow-a-v2/`.
- Create the new flow first: Power Automate → New flow → Instant cloud flow → name it `PA - Resolve Meeting Selection - v2` → trigger: **When an agent calls the flow**.

---

## Trigger setup

**T01 — Add trigger inputs.** In the trigger card, add two text inputs:

| Input | Name | Type | Required |
|---|---|---|---|
| `text_1` | InSelectedNumber | string | Yes |
| `text_3` | DateContext | string | Yes |

Save the trigger. Do not add any actions yet.

---

## Scope_FlowA — outer Scope

**S00 — Add outer Scope.** Add a Scope action immediately after the trigger. Name it `Scope_FlowA`. All subsequent actions go inside this Scope. This is the only action at the top level of the flow besides the trigger.

---

## Scope_CalendarFetch

**S01 — Add inner Scope.** Inside `Scope_FlowA`, add a Scope. Name it `Scope_CalendarFetch`.

**CA01 — Compose DateContext.**
- Type: Compose
- Name: `CA01 Compose DateContext`
- Value (Expression): `coalesce(triggerBody()?['text_3'], utcNow())`

**CA02 — Compose StartOfDay.**
- Type: Compose
- Name: `CA02 Compose StartOfDay`
- Value (Expression): `formatDateTime(if(empty(trim(coalesce(outputs('CA01_Compose_DateContext'), ''))), utcNow(), outputs('CA01_Compose_DateContext')), 'yyyy-MM-ddT00:00:00Z')`

**CA03 — Compose EndOfDay.**
- Type: Compose
- Name: `CA03 Compose EndOfDay`
- Value (Expression): `formatDateTime(if(empty(trim(coalesce(outputs('CA01_Compose_DateContext'), ''))), utcNow(), outputs('CA01_Compose_DateContext')), 'yyyy-MM-ddT23:59:59Z')`

**CA04 — Get Calendar Events.**
- Type: Office 365 Outlook — Get events (V3)
- Name: `CA04 Get Calendar Events`
- Calendar ID (Expression): `'AAMkAGY0OGU4Mzk5LWQ4NTYtNDU4MS1hY2YyLTQxOWYwZjhiMWM1ZQAuAAAAAADWkXK1vW2mQ4SwNGpyD7SzAQB8mPnOPkRmT5-MxNoNopoPAAAAAAENAAA='`
- Start time (Expression): `outputs('CA02_Compose_StartOfDay')`
- End time (Expression): `outputs('CA03_Compose_EndOfDay')`

**CA05 — Compose Raw Candidates.**
- Type: Compose
- Name: `CA05 Compose Raw Candidates`
- Value (Expression): `body('CA04_Get_Calendar_Events')?['value']`

**CA06 — Filter Exclude Leave And Periods.**
- Type: Filter Array
- Name: `CA06 Filter Exclude Leave And Periods`
- From (Expression): `outputs('CA05_Compose_Raw_Candidates')`
- Filter (advanced mode Expression):
```
@not(or(contains(toLower(coalesce(item()?['subject'],'')), 'holiday'), contains(toLower(coalesce(item()?['subject'],'')), 'leave'), contains(toLower(coalesce(item()?['subject'],'')), 'a-l'), contains(toLower(coalesce(item()?['subject'],'')), 'ooo'), contains(toLower(coalesce(item()?['subject'],'')), 'out of office'), contains(toLower(coalesce(item()?['subject'],'')), 'bank holiday'), contains(toLower(coalesce(item()?['subject'],'')), 'smarter working'), contains(toLower(coalesce(item()?['subject'],'')), 'period reminder'), contains(toLower(coalesce(item()?['subject'],'')), 'manage email'), contains(toLower(coalesce(item()?['subject'],'')), 'quiet hour')))
```

**CA07 — Compose Sorted Candidates.**
- Type: Compose
- Name: `CA07 Compose Sorted Candidates`
- Value (Expression): `sort(body('CA06_Filter_Exclude_Leave_And_Periods'), 'start')`

**Scope_CalendarFetch — corruption recovery (mandatory before next Scope):**
1. Peek Code `Scope_CalendarFetch` → paste to GitHub at `scope-peek-codes/scope-calendar-fetch.md`
2. Update `known-good-values.md` with CA01–CA07
3. Save draft → close/reopen → wait 20s → Flow Checker 0 errors → Publish

---

## Scope_CandidateResolution

**S02 — Add inner Scope.** Inside `Scope_FlowA`, after `Scope_CalendarFetch`, add a Scope. Name it `Scope_CandidateResolution`.

**CR01 — Compose Match Count.**
- Type: Compose
- Name: `CR01 Compose Match Count`
- Value (Expression): `length(outputs('CA07_Compose_Sorted_Candidates'))`

**CR02 — Compose Display Date.**
- Type: Compose
- Name: `CR02 Compose Display Date`
- Value (Expression): `formatDateTime(outputs('CA01_Compose_DateContext'), 'ddd d MMM yyyy')`

Note: pre-computed once here, referenced in MM07 — not recomputed inside the loop.

**Scope_CandidateResolution — corruption recovery (mandatory before next Scope).**

---

## Scope_NoMatch

**S03 — Add inner Scope.** Inside `Scope_FlowA`, add a Scope. Name it `Scope_NoMatch`.

**NM01 — Condition Is No Match.**
- Type: Condition
- Name: `NM01 Condition Is No Match`
- Left (Expression): `outputs('CR01_Compose_Match_Count')`
- Operator: is equal to
- Right (Expression): `0`

**Inside NM01 True branch — add these Composes in order:**

| Ref | Name | Expression |
|---|---|---|
| NM02 | Compose No Match Status | `'NO_MATCH'` |
| NM03 | Compose No Match Count | `'0'` |
| NM04 | Compose No Match List | `string('')` |
| NM05 | Compose No Match Title | `string('')` |
| NM06 | Compose No Match EventId | `string('')` |
| NM07 | Compose No Match IsRecurring | `string('')` |
| NM08 | Compose No Match SeriesMasterId | `string('')` |
| NM09 | Compose No Match JoinUrl | `string('')` |
| NM10 | Compose No Match BodyPreview | `string('')` |
| NM11 | Compose No Match EndTime | `string('')` |

Leave NM01 False branch empty.

**Scope_NoMatch — corruption recovery (mandatory before next Scope).**

---

## Scope_SingleMatch

**S04 — Add inner Scope.** Inside `Scope_FlowA`, add a Scope. Name it `Scope_SingleMatch`.

**SM01 — Condition Is Single Match.**
- Type: Condition
- Name: `SM01 Condition Is Single Match`
- Left (Expression): `outputs('CR01_Compose_Match_Count')`
- Operator: is equal to
- Right (Expression): `1`

**Inside SM01 True branch — add these actions in order:**

**SM02 — Compose Single Event.**
- Type: Compose
- Value (Expression): `outputs('CA07_Compose_Sorted_Candidates')[0]`

**SM03 — Compose Single IsRecurring.**
- Type: Compose
- Value (Expression): `if(empty(coalesce(outputs('SM02_Compose_Single_Event')?['seriesMasterId'], '')), 'false', 'true')`

**SM04 — Compose Single SeriesMasterId.**
- Type: Compose
- Value (Expression): `coalesce(outputs('SM02_Compose_Single_Event')?['seriesMasterId'], '')`

**SM05 — Compose Single Title.**
- Type: Compose
- Value (Expression): `coalesce(outputs('SM02_Compose_Single_Event')?['subject'], '')`

**SM06 — Compose Single EventId.**
- Type: Compose
- Value (Expression): `coalesce(outputs('SM02_Compose_Single_Event')?['id'], '')`

**SM07 — Compose Single EndTime.**
- Type: Compose
- Value (Expression): `coalesce(outputs('SM02_Compose_Single_Event')?['end'], '')`

⚠️ `end` is a flat string on the V4 connector. Do NOT use `?['dateTime']` on it — fails with "property selection not supported on String".

**SM08 — Compose Single BodyPreview Raw.**
- Type: Compose
- Value (Expression): `coalesce(outputs('SM02_Compose_Single_Event')?['body'], '')`

**SM09 — Compose Single BodyPreview Stripped.**
- Type: Compose
- Value (Expression): `if(equals(outputs('SM08_Compose_Single_BodyPreview_Raw'), ''), '', substring(outputs('SM08_Compose_Single_BodyPreview_Raw'), 0, max(indexOf(outputs('SM08_Compose_Single_BodyPreview_Raw'), '</body>'), 0)))`

**SM10 — Compose Single JoinUrl Anchor Index.**
- Type: Compose
- Value (Expression): `indexOf(coalesce(outputs('SM02_Compose_Single_Event')?['body'], ''), 'href="https://teams.microsoft.com/l/meetup-join/')`

**SM11 — Compose Single JoinUrl From Anchor.**
- Type: Compose
- Value (Expression): `if(equals(outputs('SM10_Compose_Single_JoinUrl_Anchor_Index'), -1), '', substring(coalesce(outputs('SM02_Compose_Single_Event')?['body'], ''), add(outputs('SM10_Compose_Single_JoinUrl_Anchor_Index'), 6), sub(length(coalesce(outputs('SM02_Compose_Single_Event')?['body'], '')), add(outputs('SM10_Compose_Single_JoinUrl_Anchor_Index'), 6))))`

**SM12 — Compose Single JoinUrl Extracted.**
- Type: Compose
- Value (Expression): `if(equals(outputs('SM11_Compose_Single_JoinUrl_From_Anchor'), ''), '', substring(outputs('SM11_Compose_Single_JoinUrl_From_Anchor'), 0, if(equals(indexOf(outputs('SM11_Compose_Single_JoinUrl_From_Anchor'), '"'), -1), length(outputs('SM11_Compose_Single_JoinUrl_From_Anchor')), indexOf(outputs('SM11_Compose_Single_JoinUrl_From_Anchor'), '"'))))`

**SM13 — Compose Single JoinUrl Final.**
- Type: Compose
- Value (Expression): `if(empty(coalesce(outputs('SM02_Compose_Single_Event')?['onlineMeeting']?['joinUrl'], '')), outputs('SM12_Compose_Single_JoinUrl_Extracted'), outputs('SM02_Compose_Single_Event')?['onlineMeeting']?['joinUrl'])`

**SM14 — Compose Single Status.**
- Type: Compose
- Value (Expression): `'MULTIPLE_MATCHES'`

⚠️ Single-match returns `MULTIPLE_MATCHES` with `matchcount=1` and empty `candidatelist`. The Topic's single-match direct-confirm behaviour depends on detecting `matchcount=1` with an empty list. Do NOT change this to a different status value.

**SM15 — Compose Single MatchCount.**
- Type: Compose
- Value (Expression): `'1'`

**SM16 — Compose Single CandidateList.**
- Type: Compose
- Value (Expression): `string('')`

Leave SM01 False branch empty.

**Scope_SingleMatch — corruption recovery (mandatory before next Scope).**

---

## Scope_MultiMatch

**S05 — Add inner Scope.** Inside `Scope_FlowA`, add a Scope. Name it `Scope_MultiMatch`.

**MM01 — Condition Is Multi Match.**
- Type: Condition
- Name: `MM01 Condition Is Multi Match`
- Left (Expression): `outputs('CR01_Compose_Match_Count')`
- Operator: greater than
- Right (Expression): `1`

**Inside MM01 True branch:**

**MM02 — Get Mapping Rows.**
- Type: SharePoint — Get items
- Name: `MM02 Get Mapping Rows`
- Site Address (Expression): `'https://jsainsbury.sharepoint.com/sites/coplt'`
- List Name (Expression): `'186b3c9f-e758-4e85-83d5-685946614a0a'`
- Filter Query (Expression): `OccurrenceDate eq '@{formatDateTime(outputs('CA01_Compose_DateContext'), 'yyyy-MM-dd')}'`
- Top Count (Expression): `50`

**MM03 — Initialise Candidate List Text.**
- Type: Initialize Variable
- Name: `MM03 Initialise Candidate List Text`
- Variable name: `varCandidateListText`
- Type: String
- Value: leave blank

⚠️ This is one of two unavoidable variables in v2. It must sit here, immediately before the loop, not at the top of the flow.

**MM04 — Build Candidate List Loop.**
- Type: For each
- Name: `MM04 Build Candidate List Loop`
- Select output (Expression): `outputs('CA07_Compose_Sorted_Candidates')`

**Inside MM04 loop:**

**MM04a — Filter Mapping Match.**
- Type: Filter Array
- Name: `MM04a Filter Mapping Match`
- From (Expression): `body('MM02_Get_Mapping_Rows')?['value']`
- Filter (advanced mode Expression): `@or(equals(item()?['SeriesMasterId'], coalesce(items('MM04_Build_Candidate_List_Loop')?['seriesMasterId'], '')), equals(item()?['MeetingId'], coalesce(items('MM04_Build_Candidate_List_Loop')?['id'], '')))`

**MM04b — Compose Status Label.**
- Type: Compose
- Name: `MM04b Compose Status Label`
- Value (Expression): `if(equals(coalesce(first(body('MM04a_Filter_Mapping_Match'))?['RecapCaptured'], false), true), ' - **Mtg Notes**', if(greater(length(body('MM04a_Filter_Mapping_Match')), 0), ' - **Captured**', ''))`

**MM04c — Append To Candidate List.**
- Type: Append to string variable
- Name: `MM04c Append To Candidate List`
- Variable name: `varCandidateListText`
- Value (Expression): `concat(string(add(indexOf(outputs('CA07_Compose_Sorted_Candidates'), items('MM04_Build_Candidate_List_Loop')), 1)), '. ', coalesce(items('MM04_Build_Candidate_List_Loop')?['subject'], 'Untitled meeting'), outputs('MM04b_Compose_Status_Label'), decodeUriComponent('%0D%0A'))`

⚠️ The `indexOf()` approach replaces the old varCandidateIndex counter entirely. Index is 0-based; adding 1 gives the 1-based display number. **Prove this expression in PA - Scratch Diagnostics before adding to the flow** — confirm `indexOf(array, item())` returns the correct position inside a foreach loop in your environment. If it does not work, fallback is: add `MM03b Initialise Candidate Index` (InitializeVariable integer, blank) before MM04, and `MM04d Increment Candidate Index` (IncrementVariable by 1) inside the loop after MM04c, referencing `variables('varCandidateIndex')` in MM04c instead.

**After MM04 loop (still inside MM01 True branch):**

| Ref | Name | Expression |
|---|---|---|
| MM05 | Compose Multi Status | `'MULTIPLE_MATCHES'` |
| MM06 | Compose Multi Match Count | `string(outputs('CR01_Compose_Match_Count'))` |
| MM07 | Compose Multi Candidate List | `concat('Meetings for ', outputs('CR02_Compose_Display_Date'), decodeUriComponent('%0D%0A'), variables('varCandidateListText'))` |
| MM08 | Compose Multi Title | `string('')` |
| MM09 | Compose Multi EventId | `string('')` |
| MM10 | Compose Multi IsRecurring | `string('')` |
| MM11 | Compose Multi SeriesMasterId | `string('')` |
| MM12 | Compose Multi JoinUrl | `string('')` |
| MM13 | Compose Multi EndTime | `string('')` |

Leave MM01 False branch empty.

**Scope_MultiMatch — corruption recovery (mandatory before next Scope).**

---

## Scope_Response

**S06 — Add inner Scope.** Inside `Scope_FlowA`, add a Scope. Name it `Scope_Response`.

**RP01 — Respond To Agent.**
- Type: Respond to a PowerApp or flow
- Name: `RP01 Respond To Agent`

Add each output field with the following Expression values:

| Field | Expression |
|---|---|
| `status` | `coalesce(outputs('NM02_Compose_No_Match_Status'), outputs('SM14_Compose_Single_Status'), outputs('MM05_Compose_Multi_Status'), 'ERROR')` |
| `matchcount` | `coalesce(outputs('NM03_Compose_No_Match_Count'), outputs('SM15_Compose_Single_MatchCount'), outputs('MM06_Compose_Multi_Match_Count'), '0')` |
| `candidatelist` | `coalesce(outputs('NM04_Compose_No_Match_List'), outputs('SM16_Compose_Single_CandidateList'), outputs('MM07_Compose_Multi_Candidate_List'), '')` |
| `meetingtitle` | `coalesce(outputs('NM05_Compose_No_Match_Title'), outputs('SM05_Compose_Single_Title'), outputs('MM08_Compose_Multi_Title'), '')` |
| `calendareventid` | `coalesce(outputs('NM06_Compose_No_Match_EventId'), outputs('SM06_Compose_Single_EventId'), outputs('MM09_Compose_Multi_EventId'), '')` |
| `isrecurring` | `coalesce(outputs('NM07_Compose_No_Match_IsRecurring'), outputs('SM03_Compose_Single_IsRecurring'), outputs('MM10_Compose_Multi_IsRecurring'), '')` |
| `seriesmasterid` | `coalesce(outputs('NM08_Compose_No_Match_SeriesMasterId'), outputs('SM04_Compose_Single_SeriesMasterId'), outputs('MM11_Compose_Multi_SeriesMasterId'), '')` |
| `onlinemeetingurl` | `coalesce(outputs('NM09_Compose_No_Match_JoinUrl'), outputs('SM13_Compose_Single_JoinUrl_Final'), outputs('MM12_Compose_Multi_JoinUrl'), '')` |
| `bodypreview` | `coalesce(outputs('NM10_Compose_No_Match_BodyPreview'), outputs('SM09_Compose_Single_BodyPreview_Stripped'), '')` |
| `endtime` | `coalesce(outputs('NM11_Compose_No_Match_EndTime'), outputs('SM07_Compose_Single_EndTime'), outputs('MM13_Compose_Multi_EndTime'), '')` |

**Scope_Response — final corruption recovery (mandatory):**
1. Peek Code `Scope_FlowA` (outer) → push to `scope-peek-codes/scope-flow-a.md`
2. Peek Code each inner Scope → push to respective files in `scope-peek-codes/`
3. Save → close/reopen → wait 20s → Flow Checker 0 errors → Publish
4. Update `known-good-values.md` with all RP01 field expressions

---

## Testing before Topic switchover

Run three direct tests against v2 before touching the Topic:

| Test | Setup | Expected result |
|---|---|---|
| No-match | Date with no meetings | `status=NO_MATCH`, all other fields empty string |
| Single-match | Date with exactly 1 meeting | Meeting details returned, `candidatelist=''`, `matchcount='1'` |
| Multi-match | Date with 2+ meetings, at least 1 with `RecapCaptured=true` | List returned with `**Mtg Notes**` label on captured meeting |

All three green → switch Topic's C2 call from v1 to v2 → run one live end-to-end capture → confirm full journey completes.

---

## Important notes

- **`indexOf()` proof required** before building MM04c. See Scope_MultiMatch section above for fallback instructions.
- **`end` field is a flat string** on the V4 calendar connector — confirmed 21 Sep 2026. Do not use `?['dateTime']` on it.
- **Single-match returns `MULTIPLE_MATCHES` status** — this is intentional. The Topic detects single-match via `matchcount=1` with empty `candidatelist`. Do not change.
- **coalesce() in RP01** works because only one branch's Composes will have executed — the others will be null (skipped). `coalesce()` returns the first non-null value, which will always be the correct branch output.
- **varCandidateListText** is the only variable in v2 (plus optionally varCandidateIndex if indexOf fallback is needed). Everything else is Compose-based.
