# Known-good values — master restore reference (maintained, update after every confirmed change)

## Purpose

The recurring platform-level corruption pattern (documented since early August, 10+ incidents logged as of 22 August) wipes the `value` field on `SetVariable`/`InitializeVariable`/`Compose` actions, typically ~20-26 actions at once. When it strikes, the fastest recovery path is pasting the correct expression back in from a known-good reference — not re-deriving it from scratch under time pressure.

**This document is that reference for Flow B** (`PA - Resolve OneNote Meeting Section - v2 Clean Build`). Keep it current: whenever an expression changes, update the row here in the same session, before moving on. A stale restore sheet is worse than none, since it invites pasting back an outdated value with confidence.

**Last verified against live flow**: 22 August 2026, via session Peek Code captures. See `session-2026-08-22-*.md` session notes for the full capture detail. If this doc and a dated Peek Code capture ever disagree, the dated capture is the source of truth.

---

## InitializeVariable actions (top of flow — no `value` field by design)

| Variable | Type | Value | Notes |
|---|---|---|---|
| `varFinalExistingPageSelfUrl` | string | *(none)* | Normal — no value field expected |
| `varFinalPageDecision` | string | *(none)* | Normal |
| `varFinalMatchCount` | string | *(none)* | Normal |
| `varOutStatus` | string | *(none)* | Normal — no default; set by `Set_varOutStatus` at end of flow |
| `varOutputPageLink` | string | *(none)* | Normal |
| `varOutputPageSelfUrl` | string | *(none)* | Normal |
| `varTargetSectionPagesUrl` | string | *(none)* | Normal — historically one of the actions most affected by corruption |
| `varOneNoteResolverResult` | string | *(none)* | Normal — same as above |
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
| `Set_varOutputPageLink_Existing` | `@first(body('Filter_Existing_Mapping'))?['PageWebUrl']` | 22 Aug — **updated from `varFinalExistingPageSelfUrl` (link-format bug fix, AMEND-2026-08-22-006)** |
| `Set_varOutputPageLink_Created_OneOff` | `@outputs('Create_Page_OneOff')?['body']?['links']?['oneNoteWebUrl']?['href']` | 22 Aug |
| `Set_varOutStatus` | See Compose section below | 22 Aug — now a six-value expression, not a literal |

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
| `Compose_UpdateHtmlFragment` | `@concat('<hr><h2>Automated update</h2><p><strong>Updated by:</strong> Meeting Capture Agent</p><p><strong>Update note:</strong> Meeting details were refreshed by the automation. Existing human-entered notes were preserved below.</p>', triggerBody()?['text_3'])` | 22 Aug — Issue #2 fix |
| `Compose_RealExistingPageId` | `@if(greater(length(body('Filter_Pages_By_Title')), 0), first(body('Filter_Pages_By_Title'))?['id'], '')` | 22 Aug — **FB-04b** — now reads from `Filter_Pages_By_Title` output, not raw section pages |
| `Compose_MappingWriteSucceeded` | `@if(equals(outputs('Create_Mapping_Item_Recurring')?['statusCode'], 201), 'true', 'false')` | 22 Aug — new action, runAfter all four statuses |
| `Compose_MappingWriteSucceeded_OneOff` | `@if(equals(outputs('Create_Mapping_Item_OneOff')?['statusCode'], 201), 'true', 'false')` | 22 Aug — new action, runAfter all four statuses |
| `Compose_SectionMatchCount_Recurring` | `@string(length(body('Filter_OneNote_Section_Recurring')))` | 22 Aug — new action |
| `Compose_SectionMatchCount_OneOff` | `@string(length(body('Filter_OneNote_Section_OneOff')))` | 22 Aug — new action |

### Compose_SafeSectionName (all three instances)

All three have the same pattern — differs only in source reference. **Copy verbatim, do not retype.**

**`Compose_SafeSectionName`** (recurring CREATE path, references `Compose_SectionDisplayName`):
```
@if(empty(trim(coalesce(outputs('Compose_SectionDisplayName'), ''))), 'Mtg - Untitled Meeting', concat('Mtg - ', substring(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(outputs('Compose_SectionDisplayName'), '/', '-'), ':', '-'), '&', 'and'), '?', ''), '*', ''), '<', ''), '>', ''), '"', ''), '|', ''), '#', ''), '''', ''), '%', ''), '~', ''), 0, min(43, length(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(outputs('Compose_SectionDisplayName'), '/', '-'), ':', '-'), '&', 'and'), '?', ''), '*', ''), '<', ''), '>', ''), '"', ''), '|', ''), '#', ''), '''', ''), '%', ''), '~', ''))))))  
```

**`Compose_SafeSectionName_ExistingBranch`** (existing-page path, references `Compose_SectionDisplayName_ExistingBranch`):
```
@if(empty(trim(coalesce(outputs('Compose_SectionDisplayName_ExistingBranch'), ''))), 'Mtg - Untitled Meeting', concat('Mtg - ', substring(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(outputs('Compose_SectionDisplayName_ExistingBranch'), '/', '-'), ':', '-'), '&', 'and'), '?', ''), '*', ''), '<', ''), '>', ''), '"', ''), '|', ''), '#', ''), '''', ''), '%', ''), '~', ''), 0, min(43, length(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(outputs('Compose_SectionDisplayName_ExistingBranch'), '/', '-'), ':', '-'), '&', 'and'), '?', ''), '*', ''), '<', ''), '>', ''), '"', ''), '|', ''), '#', ''), '''', ''), '%', ''), '~', ''))))))  
```

**`FB-F01_—_Compose_Input_MeetingTitle_(one-off)`** (one-off path, references `triggerBody()?['text_1']`):
```
@if(empty(trim(coalesce(triggerBody()?['text_1'], ''))), 'Mtg - Untitled Meeting', concat('Mtg - ', substring(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(triggerBody()?['text_1'], '/', '-'), ':', '-'), '&', 'and'), '?', ''), '*', ''), '<', ''), '>', ''), '"', ''), '|', ''), '#', ''), '''', ''), '%', ''), '~', ''), 0, min(43, length(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(triggerBody()?['text_1'], '/', '-'), ':', '-'), '&', 'and'), '?', ''), '*', ''), '<', ''), '>', ''), '"', ''), '|', ''), '#', ''), '''', ''), '%', ''), '~', ''))))))  
```

### Compose_SafePageTitle (both instances)

**`Compose_SafePageTitle`** (recurring create path):
```
@if(empty(trim(coalesce(triggerBody()?['text_1'], ''))), 'Untitled Meeting', concat(substring(replace(replace(replace(replace(triggerBody()?['text_1'], '&', 'and'), '<', ''), '>', ''), '"', ''), 0, min(150, length(replace(replace(replace(replace(triggerBody()?['text_1'], '&', 'and'), '<', ''), '>', ''), '"', '')))), if(empty(coalesce(triggerBody()?['text_5'], '')), '', concat(' - ', formatDateTime(triggerBody()?['text_5'], 'd MMM yyyy')))))
```

**`Compose_SafePageTitle_OneOff`** (one-off path, identical expression):
```
@if(empty(trim(coalesce(triggerBody()?['text_1'], ''))), 'Untitled Meeting', concat(substring(replace(replace(replace(replace(triggerBody()?['text_1'], '&', 'and'), '<', ''), '>', ''), '"', ''), 0, min(150, length(replace(replace(replace(replace(triggerBody()?['text_1'], '&', 'and'), '<', ''), '>', ''), '"', '')))), if(empty(coalesce(triggerBody()?['text_5'], '')), '', concat(' - ', formatDateTime(triggerBody()?['text_5'], 'd MMM yyyy')))))
```

### Filter_Pages_By_Title

```
@contains(item()?['title'], formatDateTime(triggerBody()?['text_5'], 'd MMM yyyy'))
```
(FB-04a — `where` clause only; `from` = `@outputs('Get_Pages_In_Section_Existing_Branch')?['body']?['value']`)

### Set_varOutStatus (six-value expression — 22 Aug, AMEND-2026-08-22-002)

```
@if(and(contains(createArray('Created','Updated','UpdatedAppend'), variables('varPageAction')), equals(coalesce(outputs('Compose_MappingWriteSucceeded'), outputs('Compose_MappingWriteSucceeded_OneOff'), 'true'), 'true')), 'SUCCESS', if(and(contains(createArray('Created','Updated','UpdatedAppend'), variables('varPageAction')), equals(coalesce(outputs('Compose_MappingWriteSucceeded'), outputs('Compose_MappingWriteSucceeded_OneOff'), 'true'), 'false')), 'PARTIAL_SUCCESS', if(and(equals(toLower(string(triggerBody()?['text'])), 'true'), empty(variables('varOneNoteResolverResult'))), 'RECURRING_SETUP_REQUIRED', if(empty(variables('varTargetSectionPagesUrl')), 'SETUP_SECTION_NOT_FOUND', if(or(greater(int(coalesce(outputs('Compose_SectionMatchCount_Recurring'), '0')), 1), greater(int(coalesce(outputs('Compose_SectionMatchCount_OneOff'), '0')), 1)), 'SETUP_SECTION_AMBIGUOUS', 'ERROR')))))
```

---

## SharePoint connector actions (key parameters)

| Action | Key parameters | Last confirmed |
|---|---|---|
| `Get_items` | `dataset: https://jsainsbury.sharepoint.com/sites/coplt`, `table: 186b3c9f-e758-4e85-83d5-685946614a0a` (GUID confirmed correct 22 Aug) | 22 Aug |
| `Create_Mapping_Item_Recurring` | `dataset: https://jsainsbury.sharepoint.com/sites/coplt`, `table: 186b3c9f-e758-4e85-83d5-685946614a0a`, fields: Title=Mapping, SeriesMasterId, MeetingTitle, SectionPagesUrl, Status/Value=Active, OccurrenceDate | 22 Aug — **new native connector, replaces raw HTTP POST** |
| `Create_Mapping_Item_OneOff` | Same dataset/table, fields: Title=Mapping, MeetingId, MeetingTitle, SectionPagesUrl, Status/Value=Active (no SeriesMasterId or OccurrenceDate) | 22 Aug — **new native connector** |
| `HTTP_Update_SP_PageSelfUrl` (recurring) | `uri: _api/web/lists/GetByTitle('RecurringMeetingSectionMap')/items(@{if(greater(length(body('Filter_Existing_Mapping')),0), first(body('Filter_Existing_Mapping'))?['ID'], body('Create_Mapping_Item_Recurring')?['ID'])})` | 22 Aug — updated to reference `Create_Mapping_Item_Recurring` |
| `OF09b_—_HTTP_Update_SP_PageSelfUrl_(OneOff)` | `uri: _api/web/lists/GetByTitle('RecurringMeetingSectionMap')/items(@{if(greater(length(body('OF01_—_Filter_Existing_Mapping_OneOff')),0), first(body('OF01_—_Filter_Existing_Mapping_OneOff'))?['ID'], body('Create_Mapping_Item_OneOff')?['ID'])})` | 22 Aug — updated to reference `Create_Mapping_Item_OneOff` |

---

## How to use this during a corruption incident

1. Confirm which actions actually lost their value (Peek Code each suspect action, or use Flow Checker to spot blank required fields — note Flow Checker has been shown to miss some blanked values, so don't rely on it alone).
2. Cross-check this table for the correct expression. If this table's "Last confirmed" date predates the most recent session, also check the latest dated session note in `12-phase-2-validation/` for any changes made since.
3. Paste back exactly — do not retype from memory, since several of these expressions are long and a small typo (a missing `?`, wrong quote type) can reintroduce a different bug.
4. Save as draft, run Flow Checker, then **explicitly Publish** before testing — draft-only saves have caused false-negative retests before.
5. Update this doc's "Last verified" date if anything here needed correcting during recovery.

---
*Last updated 22 August 2026. Supersedes the 20 August version. Keep both this table and the latest dated Peek Code capture in sync — update this table whenever an expression is deliberately changed, not just after corruption recovery.*
