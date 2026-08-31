# Known-good values — master restore reference (maintained, update after every confirmed change)

## Purpose

The recurring platform-level corruption pattern (11+ incidents as of 23 August) wipes the `value` field on `SetVariable`/`InitializeVariable`/`Compose` actions, typically 20-26 actions at once. When it strikes, the fastest recovery path is pasting the correct expression back in from this reference.

**This document covers Flow B** (`PA - Resolve OneNote Meeting Section - v2 Clean Build`). Keep it current: update whenever an expression changes, before moving on.

**Last verified against live flow:** 31 August 2026 (Stage 1 safety net built and gated — S1W01–S1W05 write-back chain added, `varOneOffMappingId` initialised, `S1_Filter_Pages_By_Title_PreCreate` null-guarded).

---

## ⚠️ Correction log

**23 Aug 2026:** The `Set_varOutStatus` expression below previously had **one extra trailing closing parenthesis**, which was pasted verbatim during a 21-action corruption recovery and caused a `TemplateValidationError` (`expected token 'EndOfData' and actual 'RightParenthesis'`) on save. The expression in this document is now corrected and paren-balance-verified (46 open / 46 close). Always verify paren balance before pasting complex nested expressions if a similar error recurs.

---

## InitializeVariable actions (top of flow — no `value` field by design)

| Variable | Type | Value | Notes |
|---|---|---|---|
| `varFinalExistingPageSelfUrl` | string | *(none)* | Normal |
| `varFinalPageDecision` | string | *(none)* | Normal |
| `varFinalMatchCount` | string | *(none)* | Normal |
| `varOutStatus` | string | *(none)* | No default — set by `Set_varOutStatus` at end of flow |
| `varOutputPageLink` | string | *(none)* | Normal |
| `varOutputPageSelfUrl` | string | *(none)* | Normal |
| `varTargetSectionPagesUrl` | string | *(none)* | Normal |
| `varOneNoteResolverResult` | string | *(none)* | Normal |
| `varPageAction` | string | *(none)* | Normal |
| `varOneOffMappingId` | string | *(none)* | Added 31 Aug — stores ID of newly created one-off mapping row for use by S1W05 across branch boundary |

---

## SetVariable actions — recurring/mapping-exists branch

| Action | Value | Last confirmed |
|---|---|---|
| `varFinalExistingPageSelfUrl_1` | `@outputs('Compose_ExistingPageSelfUrl')` | 23 Aug |
| `varFinalPageDecision_1` | `@outputs('Compose_PageDecision')` | 23 Aug |
| `varFinalMatchCount_1` | `@string(outputs('Compose_Match_Count'))` | 23 Aug |
| `Set_varOneNoteResolverResult_ExistingMapping` | `ExistingMapping` (literal) | 22 Aug |
| `Set_varTargetSectionPagesUrl_ExistingMapping` | `@first(body('Filter_Existing_Mapping'))?['SectionPagesUrl']` | 23 Aug |
| `varTargetSectionPagesUrl_1` | `@items('Apply_to_each')?['pagesUrl']` | 23 Aug |
| `varOneNoteResolverResult_1` | `ExistingSection` (literal) | 23 Aug |
| `varTargetSectionPagesUrl_2` | `@outputs('Create_Section_Recurring')?['body']?['pagesUrl']` | 23 Aug |
| `varOneNoteResolverResult_2` | `CreatedSection` (literal) | 23 Aug |

## SetVariable actions — one-off branch

| Action | Value | Last confirmed |
|---|---|---|
| `Set_varTargetSectionPagesUrl_OneOff_Exists` | `@items('For_each_1')?['pagesUrl']` | 23 Aug |
| `Set_varOneNoteResolverResult_Exists_OneOff` | `ExistingSection` (literal) | 23 Aug |
| `Set_varTargetSectionPagesUrl_OneOff_Created` | `@outputs('Create_Section_OneOff')?['body']?['pagesUrl']` | 23 Aug |
| `Set_varOneNoteResolverResult_Created_OneOff` | `CreatedSection` (literal) | 23 Aug |
| `OF05a — Set varFinalExistingPageSelfUrl (OneOff)` | `@outputs('OF02_—_Compose_ExistingPageSelfUrl_OneOff')` | 23 Aug |
| `OF05b — Set varFinalPageDecision (OneOff)` | `@outputs('OF03_—_Compose_PageDecision_OneOff')` | 23 Aug |
| `OF05c — Set varFinalMatchCount (OneOff)` | `@string(outputs('OF04_—_Compose_Match_Count_OneOff'))` | 23 Aug |

## SetVariable actions — page creation / update branch

| Action | Value | Last confirmed |
|---|---|---|
| `Set_varPageAction_Created` | `Created` (literal) | 23 Aug |
| `Set_varOutputPageSelfUrl_Created` | `@outputs('Compose_PageSelfUrl_Created')` | 23 Aug |
| `Set_varOutputPageLink_Created` | `@outputs('Create_OneNote_Page')?['body']?['links']?['oneNoteWebUrl']?['href']` | 23 Aug |
| `Set_varPageAction_Created_OneOff` | `Created` (literal) | 23 Aug |
| `Set_varOutputPageSelfUrl_Created_OneOff` | `@outputs('Compose_PageSelfUrl_Created')` | 23 Aug |
| `Set_varOutputPageLink_Created_OneOff_Gate` | `@outputs('Create_OneNote_Page')?['body']?['links']?['oneNoteWebUrl']?['href']` | 23 Aug |
| `Set_varPageAction_ExistsNoCreate` | `Updated` (literal) | 23 Aug |
| `Set_varOutputPageSelfUrl_Existing` | `@variables('varFinalExistingPageSelfUrl')` | 23 Aug |
| `Set_varPageAction_UpdatedAppend` | `Updated` (literal) | 23 Aug |
| `Set_varOutputPageLink_Existing` | `@first(coalesce(body('Filter_Existing_Mapping'), body('OF01_—_Filter_Existing_Mapping_OneOff'), createArray()))?['PageWebUrl']` | 23 Aug |
| `Set_varOutputPageLink_Created_OneOff` | `@outputs('Create_Page_OneOff')?['body']?['links']?['oneNoteWebUrl']?['href']` | 23 Aug |

## SetVariable actions — Stage 1 safety net (added 31 Aug)

These actions live inside the True branch of `S1_Condition_Title_Safety_Check`, after `S1_Set_varOutputPageLink`.

| Action | Value | Last confirmed |
|---|---|---|
| `S1_Set_varPageAction_UpdatedAppend` | `UpdatedAppend` (literal) | 31 Aug |
| `S1_Set_varOutputPageSelfUrl` | `@first(body('S1_Filter_Pages_By_Title_PreCreate'))?['self']` | 31 Aug |
| `S1_Set_varOutputPageLink` | `@first(body('S1_Filter_Pages_By_Title_PreCreate'))?['links']?['oneNoteWebUrl']?['href']` | 31 Aug |
| `S1_Set_varOneOffMappingId` | `@string(body('S1_Create_Mapping_Item_OneOff')?['ID'])` | 31 Aug |

---

## Key Compose actions

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

## Stage 1 Compose actions (added 31 Aug)

| Action | Value | Last confirmed |
|---|---|---|
| `S1_Compose_UpdateHtmlFragment` | `@concat('<hr><h2>Automated update</h2><p><strong>Updated by:</strong> Meeting Capture Agent</p><p><strong>Update note:</strong> A page with a matching title already existed in OneNote with no corresponding mapping row. The automation appended below rather than creating a duplicate page.</p>', triggerBody()?['text_3'])` | 31 Aug |
| `S1_Compose_FoundPageId` | `@if(greater(length(body('S1_Filter_Pages_By_Title_PreCreate')), 0), first(body('S1_Filter_Pages_By_Title_PreCreate'))?['id'], '')` | 31 Aug |

### Compose_SafeSectionName (all three instances)

**`Compose_SafeSectionName`** (recurring CREATE path):
```
@if(empty(trim(coalesce(outputs('Compose_SectionDisplayName'), ''))), 'Mtg - Untitled Meeting', concat('Mtg - ', substring(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(outputs('Compose_SectionDisplayName'), '/', '-'), ':', '-'), '&', 'and'), '?', ''), '*', ''), '<', ''), '>', ''), '"', ''), '|', ''), '#', ''), '''', ''), '%', ''), '~', ''), 0, min(43, length(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(outputs('Compose_SectionDisplayName'), '/', '-'), ':', '-'), '&', 'and'), '?', ''), '*', ''), '<', ''), '>', ''), '"', ''), '|', ''), '#', ''), '''', ''), '%', ''), '~', ''))))))
```

**`Compose_SafeSectionName_ExistingBranch`**:
```
@if(empty(trim(coalesce(outputs('Compose_SectionDisplayName_ExistingBranch'), ''))), 'Mtg - Untitled Meeting', concat('Mtg - ', substring(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(outputs('Compose_SectionDisplayName_ExistingBranch'), '/', '-'), ':', '-'), '&', 'and'), '?', ''), '*', ''), '<', ''), '>', ''), '"', ''), '|', ''), '#', ''), '''', ''), '%', ''), '~', ''), 0, min(43, length(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(outputs('Compose_SectionDisplayName_ExistingBranch'), '/', '-'), ':', '-'), '&', 'and'), '?', ''), '*', ''), '<', ''), '>', ''), '"', ''), '|', ''), '#', ''), '''', ''), '%', ''), '~', ''))))))
```

**`FB-F01_—_Compose_Input_MeetingTitle_(one-off)`**:
```
@if(empty(trim(coalesce(triggerBody()?['text_1'], ''))), 'Mtg - Untitled Meeting', concat('Mtg - ', substring(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(triggerBody()?['text_1'], '/', '-'), ':', '-'), '&', 'and'), '?', ''), '*', ''), '<', ''), '>', ''), '"', ''), '|', ''), '#', ''), '''', ''), '%', ''), '~', ''), 0, min(43, length(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(triggerBody()?['text_1'], '/', '-'), ':', '-'), '&', 'and'), '?', ''), '*', ''), '<', ''), '>', ''), '"', ''), '|', ''), '#', ''), '''', ''), '%', ''), '~', ''))))))
```

### Compose_SafePageTitle (both instances)

```
@if(empty(trim(coalesce(triggerBody()?['text_1'], ''))), 'Untitled Meeting', concat(substring(replace(replace(replace(replace(triggerBody()?['text_1'], '&', 'and'), '<', ''), '>', ''), '"', ''), 0, min(150, length(replace(replace(replace(replace(triggerBody()?['text_1'], '&', 'and'), '<', ''), '>', ''), '"', '')))), if(empty(coalesce(triggerBody()?['text_5'], '')), '', concat(' - ', formatDateTime(triggerBody()?['text_5'], 'd MMM yyyy')))))
```

### S1_Filter_Pages_By_Title_PreCreate where clause (null-guarded, 31 Aug)

```
@if(
  empty(coalesce(triggerBody()?['text_5'], '')),
  equals(coalesce(item()?['title'], ''), outputs('Compose_SafePageTitle')),
  contains(coalesce(item()?['title'], ''), formatDateTime(triggerBody()?['text_5'], 'd MMM yyyy'))
)
```

### Filter_Existing_Mapping where clause (UJ4b guard added 22 Aug evening)

```
@and(not(empty(triggerBody()?['text_2'])), equals(item()?['SeriesMasterId'],triggerBody()?['text_2']),equals(item()?['OccurrenceDate'],triggerBody()?['text_5']))
```

### Filter_Pages_By_Title where clause

```
@contains(item()?['title'], formatDateTime(triggerBody()?['text_5'], 'd MMM yyyy'))
```

### Set_varOutStatus (seven-value expression — CORRECTED 23 Aug, paren-balance verified: 46 open / 46 close)

**⚠️ Previous version of this doc had one extra trailing `)` — do not use any earlier copy of this expression from chat history or old exports. This is the verified-correct version:**

```
@if(and(contains(createArray('Created','Updated','UpdatedAppend'), variables('varPageAction')), equals(coalesce(outputs('Compose_MappingWriteSucceeded'), outputs('Compose_MappingWriteSucceeded_OneOff'), 'true'), 'true')), 'SUCCESS', if(and(contains(createArray('Created','Updated','UpdatedAppend'), variables('varPageAction')), equals(coalesce(outputs('Compose_MappingWriteSucceeded'), outputs('Compose_MappingWriteSucceeded_OneOff'), 'true'), 'false')), 'PARTIAL_SUCCESS', if(and(equals(toLower(string(triggerBody()?['text'])), 'true'), empty(variables('varOneNoteResolverResult'))), 'RECURRING_SETUP_REQUIRED', if(empty(variables('varTargetSectionPagesUrl')), 'SETUP_SECTION_NOT_FOUND', if(or(greater(int(coalesce(outputs('Compose_SectionMatchCount_Recurring'), '0')), 1), greater(int(coalesce(outputs('Compose_SectionMatchCount_OneOff'), '0')), 1)), 'SETUP_SECTION_AMBIGUOUS', if(and(empty(variables('varPageAction')), contains(createArray('ExistingMapping','ExistingSection'), variables('varOneNoteResolverResult'))), 'STALE_MAPPING', 'ERROR'))))))
```

### Condition_Section_Exists_Recurring (UJ4a — updated 22 Aug evening)

Outer condition expression:
```
@equals(outputs('Compose_Section_Match_Count_Recurring'), 1)
```

Nested `Condition_Section_Count_Is_Zero` (inside else branch) expression — simple mode serialisation:
```json
"equals": [
  "@outputs('Compose_Section_Match_Count_Recurring')",
  0
]
```
True branch: `Create_Section_Recurring` → `varTargetSectionPagesUrl_2` → `varOneNoteResolverResult_2 = CreatedSection`
False branch: empty — flow continues with blank vars, `OutStatus` = `SETUP_SECTION_AMBIGUOUS`

---

## SharePoint connector actions

| Action | Key parameters | Last confirmed |
|---|---|---|
| `Get_items` | `dataset: https://jsainsbury.sharepoint.com/sites/coplt`, `table: 186b3c9f-e758-4e85-83d5-685946614a0a` | 22 Aug |
| `Create_Mapping_Item_Recurring` | `PostItem`, fields: Title=Mapping, SeriesMasterId, MeetingTitle, SectionPagesUrl, Status/Value=Active, OccurrenceDate | 22 Aug |
| `Create_Mapping_Item_OneOff` | `PostItem`, fields: Title=Mapping, MeetingId, MeetingTitle, SectionPagesUrl, Status/Value=Active | 22 Aug |
| `HTTP_Update_SP_PageSelfUrl` | MERGE, URI references `body('Create_Mapping_Item_Recurring')?['ID']` | 22 Aug |
| `OF09b_—_HTTP_Update_SP_PageSelfUrl_(OneOff)` | MERGE, URI references `body('Create_Mapping_Item_OneOff')?['ID']` | 22 Aug |

## Stage 1 SharePoint connector actions (added 31 Aug)

| Action | Key parameters | Notes |
|---|---|---|
| `S1_Create_Mapping_Item_OneOff` | `PostItem`, fields: Title=Mapping, MeetingTitle=`@outputs('FB-F01_—_Compose_Input_MeetingTitle_(one-off)')`, SectionPagesUrl=`@variables('varTargetSectionPagesUrl')`, Status/Value=Active, MeetingId=`@triggerBody()?['text_4']` | PageSelfUrl/PageWebUrl left blank — written by S1W05 via MERGE |
| `S1_HTTP_Update_SP_Mapping_Recurring` | MERGE, URI: `_api/web/lists/GetByTitle('RecurringMeetingSectionMap')/items(@{if(greater(length(body('Filter_Existing_Mapping')),0), first(body('Filter_Existing_Mapping'))?['ID'], body('Create_Mapping_Item_Recurring')?['ID'])})` | Body: `{"PageSelfUrl": "@{variables('varOutputPageSelfUrl')}", "PageWebUrl": "@{variables('varOutputPageLink')}"}` |
| `S1_HTTP_Update_SP_Mapping_OneOff` | MERGE, URI: `_api/web/lists/GetByTitle('RecurringMeetingSectionMap')/items(@{if(greater(length(body('OF01_—_Filter_Existing_Mapping_OneOff')),0), first(body('OF01_—_Filter_Existing_Mapping_OneOff'))?['ID'], variables('varOneOffMappingId'))})` | Body: `{"PageSelfUrl": "@{variables('varOutputPageSelfUrl')}", "PageWebUrl": "@{variables('varOutputPageLink')}"}` |

---

## ✅ BUG-01 RESOLVED — 23 Aug 2026 (second-occurrence overwrite/collision)

**Symptom:** capturing a second occurrence of a recurring series either overwrote the first page's mapping or failed outright with a duplicate-value error.

**Root cause (confirmed, three contributing factors, all now fixed):**

1. **`varFinalMatchCount_1`/`varFinalPageDecision_1`/`varFinalExistingPageSelfUrl_1` corruption** — these were blanked by the platform corruption pattern, causing `Condition_Mapping_Exists` to always evaluate as "no existing mapping" regardless of what `Filter_Existing_Mapping` actually found. Fixed by restoring the three expressions above (23 Aug).
2. **`Set_varOutStatus` paren-balance typo** — blocked publishing entirely after the above fix; corrected (23 Aug, see Correction log above).
3. **Root structural cause: the `SeriesMasterId` column on `RecurringMeetingSectionMap` had "Enforce unique values" = Yes.** This SharePoint list-level constraint meant *any* second row for the same series — regardless of `OccurrenceDate` — would be rejected by SharePoint with a `duplicate values were found in the following field(s): [SeriesMasterId]` error, even once the flow's own logic was working correctly. **Fixed by setting Enforce unique values → No** on the `SeriesMasterId` column. The flow's own `Filter_Existing_Mapping` where-clause already prevents true duplicate rows at the logic layer.

**Validated 23 Aug:** clean list + clean OneNote section, captured occurrence 1 (24 Aug) → 1 SharePoint row, 1 OneNote page. Captured occurrence 2 (31 Aug, same series) → 2nd SharePoint row created successfully, 2nd OneNote page created successfully, neither overwrote the other.

## ✅ Data-integrity issue — orphaned/skeleton mapping rows — mitigated, UJ3b still open

Rows in `RecurringMeetingSectionMap` can end up with `Title`/`SeriesMasterId`/`MeetingTitle`/`Status`/`OccurrenceDate` populated but `SectionPagesUrl` (and other Section/Page fields) blank, if an earlier run failed after creating the mapping row but before completing the OneNote page/section creation. **UJ3b (automatic stale-row cleanup) remains not built** — until built, orphaned rows must be manually deleted from the SharePoint list before retesting an affected series.

---

## How to use during a corruption incident

1. Confirm which actions lost their value (Peek Code + Flow Checker — note Flow Checker misses some blanked values).
2. Cross-check this table. If "Last confirmed" predates the most recent session note, check that session note for any subsequent changes.
3. Paste back exactly — do not retype from memory.
4. For long nested expressions (e.g. `Set_varOutStatus`), verify parenthesis balance before pasting if the flow rejects the save with a `TemplateValidationError`.
5. Save draft, run Flow Checker, then Publish before testing.
6. Update this doc's "Last verified" date if anything needed correcting.

---
*Last updated 31 August 2026. Supersedes all prior versions.*
