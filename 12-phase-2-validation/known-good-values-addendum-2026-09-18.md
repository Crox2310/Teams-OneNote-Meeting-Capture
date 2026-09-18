# Known-good values — addendum, 18 September 2026

**Read alongside** `known-good-values-flow-a-reference.md` (Flow A) and `known-good-values-master-reference.md` (Flow B). This addendum records every value changed or added on 18 Sep 2026. Where it conflicts with the canonical docs, **this file wins** until the two are merged.

---

## Flow A — PA - Resolve Meeting Selection - v1 Clean Build

| Action | Value | Notes |
|---|---|---|
| `FA33A_Set_varCandidateListText_Empty` | `@string('')` | Re-restored 18 Sep (3rd wipe of this pair) |
| `FA34A_Set_varCandidateIndex_One` | `1` | Re-restored 18 Sep |
| `FA28C_Compose_OutEndTime` (NEW, after FA28B) | `@coalesce(outputs('FA28_Compose_SingleEvent')?['endWithTimeZone'], '')` | Single-match path |
| `FA29D_Compose_JoinUrlAnchorIndex` | `@indexOf(coalesce(outputs('FA28_Compose_SingleEvent')?['body'], ''), 'href="https://teams.microsoft.com/l/meetup-join/')` | Anchor changed from generic `teams.microsoft.com/` to meetup-join |

### FA12B_Select_Candidates — full map

```json
{
  "Title": "@coalesce(item()?['subject'], '')",
  "Start": "@coalesce(item()?['start'], '')",
  "Id": "@coalesce(item()?['id'], '')",
  "IsRecurring": "@if(empty(coalesce(item()?['seriesMasterId'], '')), 'false', 'true')",
  "SeriesMasterId": "@coalesce(item()?['seriesMasterId'], '')",
  "OnlineMeetingUrl": "@if(contains(coalesce(item()?['body'], ''), 'href=\"https://teams.microsoft.com/l/meetup-join/'), concat('https://teams.microsoft.com/l/meetup-join/', first(split(split(coalesce(item()?['body'], ''), 'href=\"https://teams.microsoft.com/l/meetup-join/')[1], '\"'))), '')",
  "BodyPreview": "@if(empty(coalesce(item()?['body'], '')), '', substring(coalesce(item()?['body'], ''), 0, min(2000, length(coalesce(item()?['body'], '')))))",
  "End": "@concat('UTC|', coalesce(item()?['endWithTimeZone'], ''))"
}
```

### FA43_Respond_to_agent — added output

| Key | Value |
|---|---|
| `endtime` | `@{coalesce(outputs('FA28C_Compose_OutEndTime'), '')}` |

Note: the Topic binds `endtime` in C2 but every path resolves EndTime via C6D, so this output is currently redundant.

---

## Flow B — PA - Resolve OneNote Meeting Section - v2 Clean Build

### Trigger (Skills) — new optional inputs

| Key | Title | Required |
|---|---|---|
| `text_6` | EndTime | No |
| `text_7` | JoinUrl | No |

`required` array = `text_1, text_2, text_3, text_4, text` (text_5/6/7 optional).

### Create_Mapping_Item_Recurring — full parameters

| Field | Value |
|---|---|
| `item/Title` | `Mapping` |
| `item/SeriesMasterId` | `@outputs('Compose_Input_SeriesMasterId')` |
| `item/MeetingTitle` | `@outputs('Compose_Input_MeetingTitle')` |
| `item/SectionPagesUrl` | `@variables('varTargetSectionPagesUrl')` |
| `item/Status/Value` | `Active` |
| `item/OccurrenceDate` | `@triggerBody()?['text_5']` |
| `item/JoinUrl` | `@trim(coalesce(triggerBody()?['text_7'], ''))` |
| `item/EndTime` | `@replace(coalesce(triggerBody()?['text_6'], ''), 'UTC|', '')` |

### 30-action restore — confirmed 18 Sep

All values in the master reference restored and confirmed on 18 Sep (recurring, one-off, D2, page-creation, OF05a–c, Set_varOutStatus). Code-view verified:

| Action | Value |
|---|---|
| `Set_varTargetSectionPagesUrl_ExistingMapping` | `@first(body('Filter_Existing_Mapping'))?['SectionPagesUrl']` |
| `varTargetSectionPagesUrl_1` | `@items('Apply_to_each')?['pagesUrl']` |
| `varTargetSectionPagesUrl_2` | `@outputs('Create_Section_Recurring')?['body']?['pagesUrl']` |
| `Set_varTargetSectionPagesUrl_D2_Exists` | `@items('For_each_D2')?['pagesUrl']` |
| `Set_varOneNoteResolverResult_Exists_D2` | `ExistingSection` |
| `Set_varTargetSectionPagesUrl_D2_Created` | `@outputs('Create_Section_D2')?['body']?['pagesUrl']` |
| `Set_varOneNoteResolverResult_Created_D2` | `CreatedSection` |

**Gap:** the SetVariables inside *Apply to each Existing Section* (existing-branch, 5 actions, incl. `Set varPageAction UpdatedAppend` and `Set varOutputPageLink Existing`) are only partially documented — capture their Code view at next opportunity.

### Connections
After the 18 Sep session loss, 5 SharePoint actions showed *Invalid connection* (Get items, Create Mapping Item Recurring, HTTP Update SP PageSelfUrl, Create Mapping Item OneOff, OF09b). Fix: *Change connection* → existing david.croxson SharePoint connection. Re-check Create Mapping Item Recurring fields afterwards (schema refetch can drop EndTime/JoinUrl).

---

## Flow C — PA - Meeting Chat Capture (Flow C)

| Action | Value |
|---|---|
| Trigger (PowerAppV2) | `text`=SeriesMasterId, `text_1`=MeetingTitle, `text_2`=OccurrenceDate, `text_3`=AllowFallback (all required) |
| `FC01_Get_Mapping_Row` `$filter` | `SeriesMasterId eq '@{triggerBody()?['text']}' and OccurrenceDate eq '@{triggerBody()?['text_2']}'`, `$top` 1 |
| `FC02_Compose_JoinUrl` | `@trim(first(body('FC01_Get_Mapping_Row')?['value'])?['JoinUrl'])` |
| `FC00a_Compose_WindowStart` | `@startOfDay(triggerBody()?['text_2'])` |
| `FC00b_Compose_WindowEnd` | `@addDays(outputs('FC00a_Compose_WindowStart'), 1)` |
| `FC05b_Filter_to_Target_Occurrence` where | `@and(greaterOrEquals(item()?['createdDateTime'], startOfDay(triggerBody()?['text_2'])), less(item()?['createdDateTime'], addDays(startOfDay(triggerBody()?['text_2']), 1)))` |
| `FC05d` AllowFallback condition | `@not(equals(toLower(coalesce(triggerBody()?['text_3'], 'true')), 'false'))` equals `true` |

---

## Flow D — PA - Auto Capture Scheduler (new reference)

| Action | Value |
|---|---|
| Trigger | Recurrence every 10 min, concurrency 1 |
| `FD01` Init `varOffsetMinutes` | Integer `5` |
| `FD03_—_Get_Mapping_Rows` | SP GetItems, OccurrenceDate = today or yesterday, top 50 |
| `FD04_—_Filter_Uncaptured_Rows` from | `@outputs('FD03_—_Get_Mapping_Rows')?['body/value']` |
| `FD04` where | `@and(not(equals(item()?['ChatCaptured'], true)), not(empty(coalesce(item()?['SeriesMasterId'], ''))), not(empty(coalesce(item()?['EndTime'], ''))))` |
| `FD06_—_For_Each_Uncaptured_Row` | foreach `@body('FD04_—_Filter_Uncaptured_Rows')`, runAfter FD04, concurrency 1 |
| `FD06c_—_Has_Offset_Elapsed` | `greaterOrEquals`: `@ticks(utcNow())` vs `@ticks(addMinutes(items('FD06_—_For_Each_Uncaptured_Row')?['EndTime'], variables('varOffsetMinutes')))` (two-field builder) |
| `FD06d_—_Run_Flow_C` (inside FD06c True) | workflowReferenceName `7b295cc2-80a5-f111-b8de-7ced8d745465`; `text`=`@items('FD06_—_For_Each_Uncaptured_Row')?['SeriesMasterId']`, `text_1`=`…?['MeetingTitle']`, `text_2`=`…?['OccurrenceDate']`, `text_3`=`true` |

Deleted: FD05, FD05b, FD06a, FD06b.

---

## Topic — Meeting Capture (v4 rebuild)

| Node | Change |
|---|---|
| C2 output binding | `endtime: Topic.EndTime` (redundant) |
| SetMultipleVariables after C2 | `Topic.EndTime = Text(Topic.EndTime)` |
| C6D | `Topic.EndTime = Text(Index(ParseJSON(Topic.CandidatesJson), Value(Topic.TopicSelectedNumber)).End)` |
| C10 inputs | `text_6: =Topic.EndTime`, `text_7: =Topic.OnlineMeetingUrl` |

**Do not** wrap EndTime in `DateTimeValue()` / `Text(..., DateTimeFormat.UTC)` — verified to produce US-format, timezone-less values.

---

## Agent

- Agent-level tool *PA - Resolve Meeting Selection - v1 Clean Build* set to **Disabled** (stale registration blocked publish with `InvalidPropertyPath`). Topic C2 still calls Flow A. Agent published 18 Sep 11:03.

---
*Created 18 September 2026.*
