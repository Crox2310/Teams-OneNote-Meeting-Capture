# Known-good values — master restore reference (maintained, update after every confirmed change)

## Purpose

The recurring platform-level corruption pattern (12+ incidents as of 7 September) wipes the `value` field on `SetVariable`/`InitializeVariable`/`Compose` actions, typically 20-27 actions at once. When it strikes, the fastest recovery path is pasting the correct expression back in from this reference.

**This document covers Flow B** (`PA - Meeting Capture - B - Resolve OneNote Section`) and Flow C (`PA - Meeting Capture - C - Chat Capture`). Keep it current: update whenever an expression changes, before moving on.

**Last verified against live flow:** 21 September 2026 (timezone fix, one-off meeting support, Flow D extensions).

---

## ⚠️ Correction log

**23 Aug 2026:** The `Set_varOutStatus` expression below previously had **one extra trailing closing parenthesis**. The expression in this document is now corrected and paren-balance-verified (46 open / 46 close).

**7 Sep 2026:** Corruption struck again — 26 actions blanked. Two non-corruption bugs also fixed in the D2 branch. See D2 branch section for details.

**18 Sep 2026:** EndTime and JoinUrl added to Flow B trigger and `Create_Mapping_Item_Recurring`. Flow D offset raised to 30 min. One-off fixed section URL added. Two new SetVariable values confirmed from Code view (`Set_varPageAction_UpdatedAppend` = `Updated`, `Set_varOutputPageLink_Existing` full expression).

**19 Sep 2026:** Flow C fully restructured — AllowFallback removed, chat always runs, new actions added. Flow B FB-F01 renamed to `Evt Mtg -`. Flow B UJ3b and existing-branch guard added. Flow B Get items Top Count set to 500.

**21 Sep 2026:**
- **Timezone fix:** Flow A FA12B `End` field changed from `endWithTimeZone` to `end` (flat UTC string, V4 connector). `end` is a flat string — do NOT use `?['dateTime']` on it (fails with "property selection not supported on String"). Flow B `Create_Mapping_Item_Recurring` EndTime no longer strips `UTC|` prefix (removed `replace()` — now just `coalesce(triggerBody()?['text_6'], '')`). Confirmed: 14:30 BST stored as 13:30 UTC.
- **Flow A corruption hit:** FA33A (`varCandidateDateListText`, value `@string('')`) and FA34A (`varCandidateIndex`, value `1`) wiped — restored.
- **Flow B one-off PostItem (D2 branch):** `Create_Mapping_Item_OneOff` now writes OccurrenceDate, JoinUrl, EndTime (was missing all three). See updated SharePoint connector actions table.
- **Flow B null guard:** `Set_varOutputPageLink_Existing` updated with null guard (see page creation/update table). `Set_varOutputPageSelfUrl_Existing` restored (was blank — corruption).
- **Flow C trigger:** `text_3` (AllowFallback) and `text_4` (MeetingId) added. `text` (SeriesMasterId) made **optional** (not in required array) to support one-off meetings passing empty string.
- **Flow C FC01:** filter now branches on empty SeriesMasterId — recurring uses `SeriesMasterId + OccurrenceDate`, one-off uses `MeetingId + OccurrenceDate`.
- **Flow D one-off support:** FD04b (filter one-off rows), FD04c (union recurring + one-off), FD06 loops FD04c, FD06d passes all five inputs including `text_4` (MeetingId) and `text` as `coalesce(..., '')` to avoid null BadRequest.
- **End-to-end confirmed:** Flow D automatically triggered Flow C for a one-off meeting; chat transcript appended to OneNote successfully.

---

## Flow B — InitializeVariable actions (top of flow — no `value` field by design)

| Variable | Type | Value | Notes |
|---|---|---|---|
| `varFinalExistingPageSelfUrl` | string | *(none)* | Normal |
| `varFinalPageDecision` | string | *(none)* | Normal |
| `varFinalMatchCount` | string | *(none)* | Normal |
| `varOutStatus` | string | *(none)* | No default |
| `varOutputPageLink` | string | *(none)* | Normal |
| `varOutputPageSelfUrl` | string | *(none)* | Normal |
| `varTargetSectionPagesUrl` | string | *(none)* | Normal |
| `varOneNoteResolverResult` | string | *(none)* | Normal |
| `varPageAction` | string | *(none)* | Normal |
| `varOneOffMappingId` | string | *(none)* | Added 31 Aug |

---

## Flow B — SetVariable actions — recurring/mapping-exists branch

| Action | Value | Last confirmed |
|---|---|---|
| `varFinalExistingPageSelfUrl_1` | `@outputs('Compose_ExistingPageSelfUrl')` | 7 Sep |
| `varFinalPageDecision_1` | `@outputs('Compose_PageDecision')` | 7 Sep |
| `varFinalMatchCount_1` | `@string(outputs('Compose_Match_Count'))` | 7 Sep |
| `Set_varOneNoteResolverResult_ExistingMapping` | `ExistingMapping` (literal) | 22 Aug |
| `Set_varTargetSectionPagesUrl_ExistingMapping` | `@first(body('Filter_Existing_Mapping'))?['SectionPagesUrl']` | 18 Sep |
| `varTargetSectionPagesUrl_1` | `@items('Apply_to_each')?['pagesUrl']` | 7 Sep |
| `varOneNoteResolverResult_1` | `ExistingSection` (literal) | 7 Sep |
| `varTargetSectionPagesUrl_2` | `@outputs('Create_Section_Recurring')?['body']?['pagesUrl']` | 7 Sep |
| `varOneNoteResolverResult_2` | `CreatedSection` (literal) | 7 Sep |

## Flow B — SetVariable actions — one-off branch

| Action | Value | Last confirmed |
|---|---|---|
| `Set_varTargetSectionPagesUrl_OneOff_Exists` | `@items('For_each_1')?['pagesUrl']` | 7 Sep |
| `Set_varOneNoteResolverResult_Exists_OneOff` | `ExistingSection` (literal) | 7 Sep |
| `Set_varTargetSectionPagesUrl_OneOff_Created` | `@outputs('Create_Section_OneOff')?['body']?['pagesUrl']` | 7 Sep |
| `Set_varOneNoteResolverResult_Created_OneOff` | `CreatedSection` (literal) | 7 Sep |
| `OF05a — Set varFinalExistingPageSelfUrl (OneOff)` | `@outputs('OF02_—_Compose_ExistingPageSelfUrl_OneOff')` | 7 Sep |
| `OF05b — Set varFinalPageDecision (OneOff)` | `@outputs('OF03_—_Compose_PageDecision_OneOff')` | 7 Sep |
| `OF05c — Set varFinalMatchCount (OneOff)` | `@string(outputs('OF04_—_Compose_Match_Count_OneOff'))` | 7 Sep |

## Flow B — SetVariable actions — page creation / update branch

| Action | Value | Last confirmed |
|---|---|---|
| `Set_varPageAction_Created` | `Created` (literal) | 7 Sep |
| `Set_varOutputPageSelfUrl_Created` | `@outputs('Compose_PageSelfUrl_Created')` | 7 Sep |
| `Set_varOutputPageLink_Created` | `@outputs('Create_OneNote_Page')?['body']?['links']?['oneNoteWebUrl']?['href']` | 7 Sep |
| `Set_varPageAction_Created_OneOff` | `Created` (literal) | 7 Sep |
| `Set_varOutputPageSelfUrl_Created_OneOff` | `@outputs('Compose_PageSelfUrl_Created')` | 7 Sep |
| `Set_varOutputPageLink_Created_OneOff_Gate` | `@outputs('Create_OneNote_Page')?['body']?['links']?['oneNoteWebUrl']?['href']` | 7 Sep |
| `Set_varPageAction_ExistsNoCreate` | `Updated` (literal) | 7 Sep |
| `Set_varOutputPageSelfUrl_Existing` | `@variables('varFinalExistingPageSelfUrl')` | 21 Sep — restored after corruption |
| `Set_varPageAction_UpdatedAppend` | `Updated` (literal) | 18 Sep — confirmed Code view |
| `Set_varOutputPageLink_Existing` | `@first(coalesce(body('Filter_Existing_Mapping'), body('OF01_—_Filter_Existing_Mapping_OneOff'), createArray()))?['PageWebUrl']` | 21 Sep — coalesce covers both recurring and one-off paths |
| `Set_varOutputPageLink_Created_OneOff` | `@outputs('Create_Page_OneOff')?['body']?['links']?['oneNoteWebUrl']?['href']` | 7 Sep |
| `Set_varTargetSectionPagesUrl_OneOffFixed` | `https://www.onenote.com/api/v1.0/myOrganization/siteCollections/b5f8860c-4772-4e8b-b340-e80ba9d490fa/sites/d814850f-59bb-4182-92b7-e25d8c6a0487/notes/sections/1-cbb5e863-2bd7-458f-9944-4fec9d20607b/pages` (text, not fx) | 18 Sep |

## Flow B — SetVariable actions — Stage 1 safety net (added 31 Aug)

| Action | Value | Last confirmed |
|---|---|---|
| `S1_Set_varPageAction_UpdatedAppend` | `UpdatedAppend` (literal) | 31 Aug |
| `S1_Set_varOutputPageSelfUrl` | `@first(body('S1_Filter_Pages_By_Title_PreCreate'))?['self']` | 31 Aug |
| `S1_Set_varOutputPageLink` | `@first(body('S1_Filter_Pages_By_Title_PreCreate'))?['links']?['oneNoteWebUrl']?['href']` | 31 Aug |
| `S1_Set_varOneOffMappingId` | `@string(body('S1_Create_Mapping_Item_OneOff')?['ID'])` | 31 Aug |

## Flow B — UJ3b stale row cleanup (added 19 Sep)

| Action | Value | Last confirmed |
|---|---|---|
| `UJ3b_Filter_Stale_Rows` from | `@body('Get_items')?['value']` | 19 Sep |
| `UJ3b_Filter_Stale_Rows` where | `@empty(item()?['SectionPagesUrl'])` | 19 Sep |
| `UJ3b_Delete_Stale_Rows` foreach | `@body('UJ3b_Filter_Stale_Rows')` | 19 Sep |
| Delete item inside loop — id | `@items('UJ3b_Delete_Stale_Rows')?['ID']` | 19 Sep |
| Delete item — dataset | `https://jsainsbury.sharepoint.com/sites/coplt` | 19 Sep |
| Delete item — table | `186b3c9f-e758-4e85-83d5-685946614a0a` | 19 Sep |

## Flow B — Existing-branch guard (added 19 Sep)

Inside `Apply_to_each_Existing_Section`, after `Compose_RealExistingPageId`.

| Action | Value | Last confirmed |
|---|---|---|
| `Guard_RealPageId_Not_Empty` condition | `@empty(outputs('Compose_RealExistingPageId'))` equals `true` | 19 Sep |
| `Guard_Create_Page_Fallback` notebookKey | `Meeting Notes\|$\|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Meeting Notes` | 19 Sep |
| `Guard_Create_Page_Fallback` sectionId | `@items('Apply_to_each_Existing_Section')?['pagesUrl']` | 19 Sep |
| `Guard_Create_Page_Fallback` pageContent | `@triggerBody()?['text_3']` | 19 Sep |
| `Guard_Update_Page_Normal` notebookKey | `Meeting Notes\|$\|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Meeting Notes` | 19 Sep |
| `Guard_Update_Page_Normal` sectionId | `@items('Apply_to_each_Existing_Section')?['pagesUrl']` | 19 Sep |
| `Guard_Update_Page_Normal` pageId | `@outputs('Compose_RealExistingPageId')` | 19 Sep |
| `Guard_Update_Page_Normal` target | `body` | 19 Sep |
| `Guard_Update_Page_Normal` action | `append` | 19 Sep |
| `Guard_Update_Page_Normal` content | `@outputs('Compose_UpdateHtmlFragment')` | 19 Sep |

---

## Flow B — D2 branch — one-off section resolution (added 5-6 Sep, fixed 7 Sep)

| Action | Value / config | Last confirmed |
|---|---|---|
| `Get_Sections_D2` | `GetSectionsInNotebook`, notebookKey: `Meeting Notes\|$\|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Meeting Notes`, connection `shared_onenote` | 7 Sep |
| `Filter_OneNote_Section_D2` where | `@equals(item()?['name'], outputs('Compose_SafeSectionName'))` — not `Compose_SafeSectionName_D2` | 7 Sep |
| `Compose_SectionMatchCount_D2` | `@length(body('Filter_OneNote_Section_D2'))` | 7 Sep |
| `Compose_Section_Match_Count_D2` | `@length(body('Filter_OneNote_Section'))` | 7 Sep |
| `Condition_Section_Exists_D2` | `@greater(outputs('Compose_Section_Match_Count_D2'), 0)` | 7 Sep |
| `Create_Section_D2` body/name | `@outputs('Compose_SafeSectionName')` — not `Compose_SafeSectionName_D2` | 7 Sep |
| `Set_varTargetSectionPagesUrl_D2_Exists` | `@items('For_each_D2')?['pagesUrl']` | 7 Sep |
| `Set_varOneNoteResolverResult_Exists_D2` | `ExistingSection` (literal) | 18 Sep — confirmed Code view |
| `Set_varTargetSectionPagesUrl_D2_Created` | `@outputs('Create_Section_D2')?['body']?['pagesUrl']` | 7 Sep |
| `Set_varOneNoteResolverResult_Created_D2` | `CreatedSection` (literal) | 7 Sep |

---

## Flow B — Key Compose actions

| Action | Value | Last confirmed |
|---|---|---|
| `Compose_Input_SeriesMasterId` | `@triggerBody()?['text_2']` | 22 Aug |
| `Compose_Input_MeetingTitle` | `@triggerBody()?['text_1']` | 22 Aug |
| `Compose_ExistingPageSelfUrl` | `@if(greater(length(body('Filter_Existing_Mapping')), 0), first(body('Filter_Existing_Mapping'))?['PageSelfUrl'], '')` | 22 Aug |
| `Compose_PageDecision` | `@if(not(empty(outputs('Compose_ExistingPageSelfUrl'))), 'PAGE_EXISTS', 'PAGE_NOT_FOUND')` | 22 Aug |
| `Compose_Match_Count` | `@length(body('Filter_Existing_Mapping'))` | 22 Aug |
| `Compose_ExistingPageId` | `@last(split(variables('varOutputPageSelfUrl'), '/'))` | 22 Aug |
| `Compose_UpdateHtmlFragment` | `@concat('<hr><h2>Automated update</h2><p><strong>Updated by:</strong> Meeting Capture Agent</p><p><strong>Update note:</strong> Meeting details were refreshed by the automation. Existing human-entered notes were preserved below.</p>', triggerBody()?['text_3'])` | 22 Aug |
| `Compose_RealExistingPageId` | `@if(greater(length(body('Filter_Pages_By_Title')), 0), first(body('Filter_Pages_By_Title'))?['id'], '')` | 22 Aug |
| `Compose_MappingWriteSucceeded` | `@if(equals(outputs('Create_Mapping_Item_Recurring')?['statusCode'], 201), 'true', 'false')` | 22 Aug |
| `Compose_MappingWriteSucceeded_OneOff` | `@if(equals(outputs('Create_Mapping_Item_OneOff')?['statusCode'], 201), 'true', 'false')` | 22 Aug |
| `Compose_SectionMatchCount_Recurring` | `@string(length(body('Filter_OneNote_Section_Recurring')))` | 22 Aug |
| `Compose_SectionMatchCount_OneOff` | `@string(length(body('Filter_OneNote_Section_OneOff')))` | 22 Aug |

### Compose_SafeSectionName — recurring CREATE path (prefix updated 19 Sep)

```
@if(empty(trim(coalesce(outputs('Compose_SectionDisplayName'), ''))), 'Mtg - Untitled Meeting', concat('Mtg - ', substring(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(outputs('Compose_SectionDisplayName'), '/', '-'), ':', '-'), '&', 'and'), '?', ''), '*', ''), '<', ''), '>', ''), '"', ''), '|', ''), '#', ''), '''', ''), '%', ''), '~', ''), 0, min(43, length(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(outputs('Compose_SectionDisplayName'), '/', '-'), ':', '-'), '&', 'and'), '?', ''), '*', ''), '<', ''), '>', ''), '"', ''), '|', ''), '#', ''), '''', ''), '%', ''), '~', ''))))))
```

Note: `Mtg -` prefix unchanged for recurring meetings.

### Compose_SafeSectionName_ExistingBranch

```
@if(empty(trim(coalesce(outputs('Compose_SectionDisplayName_ExistingBranch'), ''))), 'Mtg - Untitled Meeting', concat('Mtg - ', substring(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(outputs('Compose_SectionDisplayName_ExistingBranch'), '/', '-'), ':', '-'), '&', 'and'), '?', ''), '*', ''), '<', ''), '>', ''), '"', ''), '|', ''), '#', ''), '''', ''), '%', ''), '~', ''), 0, min(43, length(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(outputs('Compose_SectionDisplayName_ExistingBranch'), '/', '-'), ':', '-'), '&', 'and'), '?', ''), '*', ''), '<', ''), '>', ''), '"', ''), '|', ''), '#', ''), '''', ''), '%', ''), '~', ''))))))
```

### FB-F01 — Compose Input MeetingTitle (one-off) — updated 19 Sep

```
@if(empty(trim(coalesce(triggerBody()?['text_1'], ''))), 'Evt Mtg - Untitled Meeting', concat('Evt Mtg - ', substring(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(triggerBody()?['text_1'], '/', '-'), ':', '-'), '&', 'and'), '?', ''), '*', ''), '<', ''), '>', ''), '"', ''), '|', ''), '#', ''), '''', ''), '%', ''), '~', ''), 0, min(43, length(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(triggerBody()?['text_1'], '/', '-'), ':', '-'), '&', 'and'), '?', ''), '*', ''), '<', ''), '>', ''), '"', ''), '|', ''), '#', ''), '''', ''), '%', ''), '~', ''))))))
```

Note: prefix changed from `Mtg -` to `Evt Mtg -` on 19 Sep. Recurring meetings still use `Mtg -`.

### Compose_SafePageTitle (both instances)

```
@if(empty(trim(coalesce(triggerBody()?['text_1'], ''))), 'Untitled Meeting', concat(substring(replace(replace(replace(replace(triggerBody()?['text_1'], '&', 'and'), '<', ''), '>', ''), '"', ''), 0, min(150, length(replace(replace(replace(replace(triggerBody()?['text_1'], '&', 'and'), '<', ''), '>', ''), '"', '')))), if(empty(coalesce(triggerBody()?['text_5'], '')), '', concat(' - ', formatDateTime(triggerBody()?['text_5'], 'd MMM yyyy')))))
```

### Filter_Existing_Mapping where clause

```
@and(not(empty(triggerBody()?['text_2'])), equals(item()?['SeriesMasterId'],triggerBody()?['text_2']),equals(item()?['OccurrenceDate'],triggerBody()?['text_5']))
```

### Filter_Pages_By_Title where clause

```
@contains(item()?['title'], formatDateTime(triggerBody()?['text_5'], 'd MMM yyyy'))
```

### Set_varOutStatus (paren-balance verified: 46 open / 46 close)

```
@if(and(contains(createArray('Created','Updated','UpdatedAppend'), variables('varPageAction')), equals(coalesce(outputs('Compose_MappingWriteSucceeded'), outputs('Compose_MappingWriteSucceeded_OneOff'), 'true'), 'true')), 'SUCCESS', if(and(contains(createArray('Created','Updated','UpdatedAppend'), variables('varPageAction')), equals(coalesce(outputs('Compose_MappingWriteSucceeded'), outputs('Compose_MappingWriteSucceeded_OneOff'), 'true'), 'false')), 'PARTIAL_SUCCESS', if(and(equals(toLower(string(triggerBody()?['text'])), 'true'), empty(variables('varOneNoteResolverResult'))), 'RECURRING_SETUP_REQUIRED', if(empty(variables('varTargetSectionPagesUrl')), 'SETUP_SECTION_NOT_FOUND', if(or(greater(int(coalesce(outputs('Compose_SectionMatchCount_Recurring'), '0')), 1), greater(int(coalesce(outputs('Compose_SectionMatchCount_OneOff'), '0')), 1)), 'SETUP_SECTION_AMBIGUOUS', if(and(empty(variables('varPageAction')), contains(createArray('ExistingMapping','ExistingSection'), variables('varOneNoteResolverResult'))), 'STALE_MAPPING', 'ERROR'))))))
```

---

## Flow A — Key actions (21 Sep 2026)

| Action | Value | Last confirmed |
|---|---|---|
| `FA12B` `End` field | `@coalesce(item()?['end'], '')` | 21 Sep — `end` is a FLAT STRING on V4 connector, NOT nested; `item()?['end']?['dateTime']` fails |
| `FA33A` Set varCandidateDateListText Empty | `@string('')` | 21 Sep — corruption-prone; restore if blank |
| `FA34A` Set varCandidateIndex One | `1` (integer) | 21 Sep — corruption-prone; restore if blank |

---

## Flow B — SharePoint connector actions

| Action | Key parameters | Last confirmed |
|---|---|---|
| `Get_items` | dataset: `https://jsainsbury.sharepoint.com/sites/coplt`, table: `186b3c9f-e758-4e85-83d5-685946614a0a`, `$top`: 500 | 19 Sep |
| `Create_Mapping_Item_Recurring` | PostItem — Title=Mapping, SeriesMasterId, MeetingTitle, SectionPagesUrl, Status/Value=Active, OccurrenceDate, JoinUrl=`@trim(coalesce(triggerBody()?['text_7'], ''))`, EndTime=`@coalesce(triggerBody()?['text_6'], '')` | 21 Sep — `replace('UTC\|','')` removed |
| `Create_Mapping_Item_OneOff` | PostItem — Title=Mapping, MeetingId, MeetingTitle, SectionPagesUrl, Status/Value=Active, OccurrenceDate=`@triggerBody()?['text_5']`, JoinUrl=`@trim(coalesce(triggerBody()?['text_7'], ''))`, EndTime=`@coalesce(triggerBody()?['text_6'], '')` | 21 Sep — OccurrenceDate/JoinUrl/EndTime added |
| `HTTP_Update_SP_PageSelfUrl` | MERGE, URI references `body('Create_Mapping_Item_Recurring')?['ID']` | 22 Aug |
| `OF09b_—_HTTP_Update_SP_PageSelfUrl_(OneOff)` | MERGE, URI references `body('Create_Mapping_Item_OneOff')?['ID']` | 22 Aug |

---

## Flow B — Trigger inputs (current)

| Key | Title | Required |
|---|---|---|
| `text` | IsRecurring | Yes |
| `text_1` | MeetingTitle | Yes |
| `text_2` | SeriesMasterId | Yes |
| `text_3` | PageHtml | Yes |
| `text_4` | MeetingId | Yes |
| `text_5` | OccurrenceDate | No |
| `text_6` | EndTime | No |
| `text_7` | JoinUrl | No |

---

## Flow C — PA - Meeting Capture - C - Chat Capture (current as of 21 Sep)

**Note:** AllowFallback trigger input removed 19 Sep. FC05d deleted. Chat always runs at main flow level. MeetingId (text_4) added 21 Sep for one-off meeting support. SeriesMasterId (text) made optional 21 Sep.

### Trigger inputs (current)

| Key | Title | Required |
|---|---|---|
| `text` | SeriesMasterId | **No** (made optional 21 Sep — one-off meetings pass empty string) |
| `text_1` | MeetingTitle | Yes |
| `text_2` | OccurrenceDate | Yes |
| `text_3` | AllowFallback | No |
| `text_4` | MeetingId | No |

### FC01 Get Mapping Row — filter (updated 21 Sep)

Branches on whether SeriesMasterId is empty:

```
@if(empty(triggerBody()?['text']), concat('MeetingId eq ''', triggerBody()?['text_4'], ''' and OccurrenceDate eq ''', triggerBody()?['text_2'], ''''), concat('SeriesMasterId eq ''', triggerBody()?['text'], ''' and OccurrenceDate eq ''', triggerBody()?['text_2'], ''''))
```

- Empty SeriesMasterId → filter by MeetingId + OccurrenceDate (one-off path)
- Populated SeriesMasterId → filter by SeriesMasterId + OccurrenceDate (recurring path)

### Key action values

| Action | Value | Last confirmed |
|---|---|---|
| `FC00a_Compose_WindowStart` | `@startOfDay(triggerBody()?['text_2'])` | 19 Sep |
| `FC00b_Compose_WindowEnd` | `@addDays(outputs('FC00a_Compose_WindowStart'), 1)` | 19 Sep |
| `FC01_Get_Mapping_Row` `$top` | `1` | 19 Sep |
| `FC02_Compose_JoinUrl` | `@first(body('FC01_Get_Mapping_Row')?['value'])?['JoinUrl']` | 19 Sep |
| `FC03_Compose_PageSelfUrl` | `@first(body('FC01_Get_Mapping_Row')?['value'])?['PageSelfUrl']` | 19 Sep |
| `FC04_Compose_SectionPagesUrl` | `@first(body('FC01_Get_Mapping_Row')?['value'])?['SectionPagesUrl']` | 19 Sep |
| `FC05s_Compose_Insight_HTML` | `@concat('<hr><h2>Meeting Capture</h2><h3>Summary</h3>', join(body('FC05p_Format_MeetingNotes_Rows'), ''), '<h3>Action Items</h3>', join(body('FC05q_Format_ActionItems_Rows'), ''))` | 19 Sep — leading space bug fixed |
| `FC05t_Update_Notes` notebookKey | `Meeting Notes\|$\|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Master Archive Folder/Meeting Notes` | 19 Sep |
| `FC05t_Update_Notes` sectionId | `@outputs('FC04_Compose_SectionPagesUrl')` | 19 Sep |
| `FC05t_Update_Notes` pageId | `@last(split(outputs('FC03_Compose_PageSelfUrl'), '/'))` | 19 Sep |
| `FC05t_Update_Notes` target | `body` | 19 Sep |
| `FC05t_Update_Notes` action | `append` | 19 Sep |
| `FC05t_Update_Notes` content | `@outputs('FC05s_Compose_Insight_HTML')` | 19 Sep |
| `FC06_Compose_ThreadId_NEW` | `@body('FC05_Get_Online_Meeting')?['chatInfo']?['threadId']` | 19 Sep |
| `FC07_Get_Chat_Messages_NEW` URI | `@concat('https://graph.microsoft.com/v1.0/me/chats/', outputs('FC06_Compose_ThreadId_NEW'), '/messages?$top=50&$orderby=createdDateTime desc')` | 19 Sep |
| `FC07_Get_Chat_Messages_NEW` Method | `GET` | 19 Sep |
| `FC08_Filter_Real_Messages_NEW` from | `@body('FC07_Get_Chat_Messages_NEW')?['value']` | 19 Sep |
| `FC08_Filter_Real_Messages_NEW` where | `@equals(coalesce(item()?['messageType'], ''), 'message')` | 19 Sep |
| `FC09_Filter_By_Window_NEW` from | `@body('FC08_Filter_Real_Messages_NEW')` | 19 Sep |
| `FC09_Filter_By_Window_NEW` where | `@and(greaterOrEquals(ticks(coalesce(item()?['createdDateTime'], '1900-01-01T00:00:00Z')), ticks(outputs('FC00a_Compose_WindowStart'))), less(ticks(coalesce(item()?['createdDateTime'], '1900-01-01T00:00:00Z')), ticks(outputs('FC00b_Compose_WindowEnd'))))` | 19 Sep |
| `FC11_Sort_By_Time_NEW` | `@sort(body('FC10_Select_Message_Fields_NEW'), 'Time')` | 19 Sep |
| `FC12_Compose_Chat_Summary_NEW` | `@join(body('FC11B_Format_Message_Rows_NEW'), '')` | 19 Sep — heading removed to avoid duplication |
| `FC15_Update_Chat` notebookKey | `Meeting Notes\|$\|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Master Archive Folder/Meeting Notes` | 19 Sep |
| `FC15_Update_Chat` sectionId | `@outputs('FC04_Compose_SectionPagesUrl')` | 19 Sep |
| `FC15_Update_Chat` pageId | `@last(split(outputs('FC03_Compose_PageSelfUrl'), '/'))` | 19 Sep |
| `FC15_Update_Chat` target | `body` | 19 Sep |
| `FC15_Update_Chat` action | `append` | 19 Sep |
| `FC15_Update_Chat` content | `@outputs('FC12_Compose_Chat_Summary_NEW')` | 19 Sep |
| `FC16_Set_ChatCaptured` dataset | `https://jsainsbury.sharepoint.com/sites/coplt` | 19 Sep |
| `FC16_Set_ChatCaptured` table | `186b3c9f-e758-4e85-83d5-685946614a0a` | 19 Sep |
| `FC16_Set_ChatCaptured` id | `@first(body('FC01_Get_Mapping_Row')?['value'])?['ID']` | 19 Sep |
| `FC16_Set_ChatCaptured` ChatCaptured | `true` | 19 Sep |

### FC10_Select_Message_Fields_NEW map

| Key | Value |
|---|---|
| `Time` | `@coalesce(item()?['createdDateTime'], '')` |
| `Speaker` | `@coalesce(item()?['from']?['user']?['displayName'], item()?['from']?['application']?['displayName'], 'Unknown')` |
| `Html` | `@coalesce(item()?['body']?['content'], '')` |

### FC11B_Format_Message_Rows_NEW select expression

```
@concat('<p><strong>', formatDateTime(item()?['Time'], 'HH:mm'), ' - ', item()?['Speaker'], ':</strong> ', item()?['Html'], '</p>')
```

---

## Flow D — PA - Meeting Capture - D - Auto Scheduler (current as of 21 Sep)

| Action | Value | Last confirmed |
|---|---|---|
| `FD01` Init `varOffsetMinutes` | `30` | 19 Sep |
| `FD04` filter where | `@and(not(equals(item()?['ChatCaptured'], true)), not(empty(coalesce(item()?['SeriesMasterId'], ''))), not(empty(coalesce(item()?['EndTime'], ''))))` | 21 Sep |
| `FD04b` filter from | `@outputs('FD03_—_Get_Mapping_Rows')?['body/value']` | 21 Sep |
| `FD04b` filter where | `@and(not(equals(item()?['ChatCaptured'], true)), empty(coalesce(item()?['SeriesMasterId'], '')), not(empty(coalesce(item()?['MeetingId'], ''))), not(empty(coalesce(item()?['EndTime'], ''))))` | 21 Sep |
| `FD04c` Compose (union) | `@union(body('FD04_—_Filter_Uncaptured_Rows'), body('FD04b_—_Filter_OneOff_Rows'))` | 21 Sep |
| `FD06` foreach | `@outputs('FD04c_—_Union_Rows')` | 21 Sep |
| `FD06d_—_Run_Flow_C` workflowReferenceName | `7b295cc2-80a5-f111-b8de-7ced8d745465` | 18 Sep |
| `FD06d` `text` | `@coalesce(items('FD06_—_For_Each_Uncaptured_Row')?['SeriesMasterId'], '')` | 21 Sep — coalesce to '' avoids null BadRequest on one-off rows |
| `FD06d` `text_1` | `@items('FD06_—_For_Each_Uncaptured_Row')?['MeetingTitle']` | 21 Sep |
| `FD06d` `text_2` | `@items('FD06_—_For_Each_Uncaptured_Row')?['OccurrenceDate']` | 21 Sep |
| `FD06d` `text_3` | `true` (literal) | 21 Sep |
| `FD06d` `text_4` | `@items('FD06_—_For_Each_Uncaptured_Row')?['MeetingId']` | 21 Sep |

---

## Agent — Meeting Capture topic

| Node | Value | Last confirmed |
|---|---|---|
| C2 output binding `endtime` | `Topic.EndTime` | 18 Sep |
| SetMultipleVariables after C2 | `Topic.EndTime = Text(Topic.EndTime)` | 18 Sep |
| C6D EndTime assignment | `Topic.EndTime = Text(Index(ParseJSON(Topic.CandidatesJson), Value(Topic.TopicSelectedNumber)).End)` | 18 Sep |
| C10 `text_6` | `=Topic.EndTime` | 18 Sep |
| C10 `text_7` | `=Topic.OnlineMeetingUrl` | 18 Sep |

---

## Key reference values

**One-Off Meetings fixed section URL:**
```
https://www.onenote.com/api/v1.0/myOrganization/siteCollections/b5f8860c-4772-4e8b-b340-e80ba9d490fa/sites/d814850f-59bb-4182-92b7-e25d8c6a0487/notes/sections/1-cbb5e863-2bd7-458f-9944-4fec9d20607b/pages
```

**Notebook key — Master Archive (Flow C, and FC05t/FC15):**
```
Meeting Notes|$|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Master Archive Folder/Meeting Notes
```

**Notebook key — standard (most Flow B actions):**
```
Meeting Notes|$|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Meeting Notes
```

**SharePoint list GUID:** `186b3c9f-e758-4e85-83d5-685946614a0a`

**Flow IDs:**
- A: `d9d7ccf7`
- B: `ed112c88`
- C: `7b295cc2`
- D: `d5faeba7`

---

## How to use during a corruption incident

1. Confirm which actions lost their value (Code view + Flow Checker).
2. Cross-check this table. If "Last confirmed" predates the most recent session note, check that session note for any subsequent changes.
3. Paste back exactly — do not retype from memory.
4. For long nested expressions, verify parenthesis balance before pasting if the flow rejects with a `TemplateValidationError`.
5. Save draft, run Flow Checker, then Publish before testing.
6. After restoring, close and reopen the flow, wait 20s, run Flow Checker again before publishing.
7. Check `Condition_IsRecurring` Code view to confirm `varFinalPageDecision_1` = `@outputs('Compose_PageDecision')` not `""`.

---

*Last updated 21 September 2026. Supersedes all prior versions.*
