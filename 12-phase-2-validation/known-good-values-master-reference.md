# Known-good values — master restore reference (maintained, update after every confirmed change)

## Purpose

The recurring platform-level corruption pattern (10+ incidents as of 22 August) wipes the `value` field on `SetVariable`/`InitializeVariable`/`Compose` actions, typically 20-26 actions at once. When it strikes, the fastest recovery path is pasting the correct expression back in from this reference.

**This document covers Flow B** (`PA - Resolve OneNote Meeting Section - v2 Clean Build`). Keep it current: update whenever an expression changes, before moving on.

**Last verified against live flow:** 22 August 2026 (evening session). See `session-2026-08-22-evening-uj345.md` for the most recent changes.

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

---

## SetVariable actions — recurring/mapping-exists branch

| Action | Value | Last confirmed |
|---|---|---|
| `varFinalExistingPageSelfUrl_1` | `@outputs('Compose_ExistingPageSelfUrl')` | 22 Aug |
| `varFinalPageDecision_1` | `@outputs('Compose_PageDecision')` | 22 Aug |
| `varFinalMatchCount_1` | `@string(outputs('Compose_Match_Count'))` | 22 Aug |
| `Set_varOneNoteResolverResult_ExistingMapping` | `ExistingMapping` (literal) | 22 Aug |
| `Set_varTargetSectionPagesUrl_ExistingMapping` | `@first(body('Filter_Existing_Mapping'))?['SectionPagesUrl']` | 22 Aug |
| `varTargetSectionPagesUrl_1` | `@items('Apply_to_each')?['pagesUrl']` | 22 Aug |
| `varOneNoteResolverResult_1` | `ExistingSection` (literal) | 22 Aug |
| `varTargetSectionPagesUrl_2` | `@outputs('Create_Section_Recurring')?['body']?['pagesUrl']` | 22 Aug |
| `varOneNoteResolverResult_2` | `CreatedSection` (literal) | 22 Aug |

## SetVariable actions — one-off branch

| Action | Value | Last confirmed |
|---|---|---|
| `Set_varTargetSectionPagesUrl_OneOff_Exists` | `@items('For_each_1')?['pagesUrl']` | 22 Aug |
| `Set_varOneNoteResolverResult_Exists_OneOff` | `ExistingSection` (literal) | 22 Aug |
| `Set_varTargetSectionPagesUrl_OneOff_Created` | `@outputs('Create_Section_OneOff')?['body']?['pagesUrl']` | 22 Aug |
| `Set_varOneNoteResolverResult_Created_OneOff` | `CreatedSection` (literal) | 22 Aug |
| `OF05a — Set varFinalExistingPageSelfUrl (OneOff)` | `@outputs('OF02_—_Compose_ExistingPageSelfUrl_OneOff')` | 22 Aug |
| `OF05b — Set varFinalPageDecision (OneOff)` | `@outputs('OF03_—_Compose_PageDecision_OneOff')` | 22 Aug |
| `OF05c — Set varFinalMatchCount (OneOff)` | `@string(outputs('OF04_—_Compose_Match_Count_OneOff'))` | 22 Aug |

## SetVariable actions — page creation / update branch

| Action | Value | Last confirmed |
|---|---|---|
| `Set_varPageAction_Created` | `Created` (literal) | 22 Aug |
| `Set_varOutputPageSelfUrl_Created` | `@outputs('Compose_PageSelfUrl_Created')` | 22 Aug |
| `Set_varOutputPageLink_Created` | `@outputs('Create_OneNote_Page')?['body']?['links']?['oneNoteWebUrl']?['href']` | 22 Aug |
| `Set_varPageAction_Created_OneOff` | `Created` (literal) | 22 Aug |
| `Set_varOutputPageSelfUrl_Created_OneOff` | `@outputs('Compose_PageSelfUrl_Created')` | 22 Aug |
| `Set_varOutputPageLink_Created_OneOff_Gate` | `@outputs('Create_OneNote_Page')?['body']?['links']?['oneNoteWebUrl']?['href']` | 22 Aug |
| `Set_varPageAction_ExistsNoCreate` | `Updated` (literal) | 22 Aug |
| `Set_varOutputPageSelfUrl_Existing` | `@variables('varFinalExistingPageSelfUrl')` | 22 Aug |
| `Set_varPageAction_UpdatedAppend` | `Updated` (literal) | 22 Aug |
| `Set_varOutputPageLink_Existing` | `@first(body('Filter_Existing_Mapping'))?['PageWebUrl']` | 22 Aug |
| `Set_varOutputPageLink_Created_OneOff` | `@outputs('Create_Page_OneOff')?['body']?['links']?['oneNoteWebUrl']?['href']` | 22 Aug |

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

### Filter_Existing_Mapping where clause (UJ4b guard added 22 Aug evening)

```
@and(not(empty(triggerBody()?['text_2'])), equals(item()?['SeriesMasterId'],triggerBody()?['text_2']),equals(item()?['OccurrenceDate'],triggerBody()?['text_5']))
```

### Filter_Pages_By_Title where clause

```
@contains(item()?['title'], formatDateTime(triggerBody()?['text_5'], 'd MMM yyyy'))
```

### Set_varOutStatus (seven-value expression — 22 Aug evening, includes STALE_MAPPING)

```
@if(and(contains(createArray('Created','Updated','UpdatedAppend'), variables('varPageAction')), equals(coalesce(outputs('Compose_MappingWriteSucceeded'), outputs('Compose_MappingWriteSucceeded_OneOff'), 'true'), 'true')), 'SUCCESS', if(and(contains(createArray('Created','Updated','UpdatedAppend'), variables('varPageAction')), equals(coalesce(outputs('Compose_MappingWriteSucceeded'), outputs('Compose_MappingWriteSucceeded_OneOff'), 'true'), 'false')), 'PARTIAL_SUCCESS', if(and(equals(toLower(string(triggerBody()?['text'])), 'true'), empty(variables('varOneNoteResolverResult'))), 'RECURRING_SETUP_REQUIRED', if(empty(variables('varTargetSectionPagesUrl')), 'SETUP_SECTION_NOT_FOUND', if(or(greater(int(coalesce(outputs('Compose_SectionMatchCount_Recurring'), '0')), 1), greater(int(coalesce(outputs('Compose_SectionMatchCount_OneOff'), '0')), 1)), 'SETUP_SECTION_AMBIGUOUS', if(and(empty(variables('varPageAction')), contains(createArray('ExistingMapping','ExistingSection'), variables('varOneNoteResolverResult'))), 'STALE_MAPPING', 'ERROR')))))))
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
| `HTTP_Update_SP_PageSelfUrl` | URI references `body('Create_Mapping_Item_Recurring')?['ID']` | 22 Aug |
| `OF09b_—_HTTP_Update_SP_PageSelfUrl_(OneOff)` | URI references `body('Create_Mapping_Item_OneOff')?['ID']` | 22 Aug |

---

## How to use during a corruption incident

1. Confirm which actions lost their value (Peek Code + Flow Checker — note Flow Checker misses some blanked values).
2. Cross-check this table. If "Last confirmed" predates the most recent session note, check that session note for any subsequent changes.
3. Paste back exactly — do not retype from memory.
4. Save draft, run Flow Checker, then Publish before testing.
5. Update this doc's "Last verified" date if anything needed correcting.

---
*Last updated 22 August 2026 (evening). Supersedes all prior versions.*
