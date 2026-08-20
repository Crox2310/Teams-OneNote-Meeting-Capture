# Known-good values — master restore reference (maintained, update after every confirmed change)

## Purpose

The recurring platform-level corruption pattern (documented since early August, 7 incidents logged as of 20 August) wipes the `value` field on `SetVariable`/`InitializeVariable`/`Compose` actions, typically ~20-26 actions at once. When it strikes, the fastest recovery path is pasting the correct expression back in from a known-good reference — not re-deriving it from scratch under time pressure.

**This document is that reference for Flow B** (`PA - Resolve OneNote Meeting Section - v2 Clean Build`). Keep it current: whenever an expression changes, update the row here in the same session, before moving on. A stale restore sheet is worse than none, since it invites pasting back an outdated value with confidence.

**Last verified against live flow**: 20 August 2026, via container-level Peek Code capture (`flow-reference-2026-08-20-full-peek-code-capture.md`). Cross-check that doc if this one and it ever disagree — the dated capture is the source of truth; this table is a derived convenience view.

---

## InitializeVariable actions (top of flow — no `value` field by design, except `varOutStatus`)

| Variable | Type | Value | Notes |
|---|---|---|---|
| `varFinalExistingPageSelfUrl` | string | *(none)* | Normal — no value field expected |
| `varFinalPageDecision` | string | *(none)* | Normal |
| `varFinalMatchCount` | string | *(none)* | Normal |
| `varOutStatus` | string | `OK` | Set at init; also set again later via `Set_varOutStatus` before Respond — both should read `OK` |
| `varOutputPageLink` | string | *(none)* | Normal |
| `varOutputPageSelfUrl` | string | *(none)* | Normal |
| `varTargetSectionPagesUrl` | string | *(none)* | Normal — historically one of the actions most affected by corruption |
| `varOneNoteResolverResult` | string | *(none)* | Normal — same as above |
| `varPageAction` | string | *(none)* | Normal |

## SetVariable actions — recurring/mapping-exists branch

| Action | Value | Source |
|---|---|---|
| `varFinalExistingPageSelfUrl_1` | `@outputs('Compose_ExistingPageSelfUrl')` | 20 Aug capture |
| `varFinalPageDecision_1` | `@outputs('Compose_PageDecision')` | 20 Aug capture |
| `varFinalMatchCount_1` | `@string(outputs('Compose_Match_Count'))` | 20 Aug capture |
| `Set_varOneNoteResolverResult_ExistingMapping` | `ExistingMapping` (literal) | 20 Aug capture |
| `Set_varTargetSectionPagesUrl_ExistingMapping` | `@first(body('Filter_Existing_Mapping'))?['SectionPagesUrl']` | 20 Aug capture |
| `varTargetSectionPagesUrl_1` | `@items('Apply_to_each')?['pagesUrl']` | 20 Aug capture |
| `varOneNoteResolverResult_1` | `ExistingSection` (literal) | 20 Aug capture |
| `varTargetSectionPagesUrl_2` | `@outputs('Create_Section_Recurring')?['body']?['pagesUrl']` | 20 Aug capture |
| `varOneNoteResolverResult_2` | `CreatedSection` (literal) | 20 Aug capture |

## SetVariable actions — one-off branch

| Action | Value | Source |
|---|---|---|
| `Set_varTargetSectionPagesUrl_OneOff_Exists` | `@items('For_each_1')?['pagesUrl']` | 20 Aug capture |
| `Set_varOneNoteResolverResult_Exists_OneOff` | `ExistingSection` (literal) | 20 Aug capture |
| `Set_varTargetSectionPagesUrl_OneOff_Created` | `@outputs('Create_Section_OneOff')?['body']?['pagesUrl']` | 20 Aug capture |
| `Set_varOneNoteResolverResult_Created_OneOff` | `CreatedSection` (literal) | 20 Aug capture |
| `OF05a — Set varFinalExistingPageSelfUrl (OneOff)` | `@outputs('OF02_—_Compose_ExistingPageSelfUrl_OneOff')` | 20 Aug capture |
| `OF05b — Set varFinalPageDecision (OneOff)` | `@outputs('OF03_—_Compose_PageDecision_OneOff')` | 20 Aug capture |
| `OF05c — Set varFinalMatchCount (OneOff)` | `@string(outputs('OF04_—_Compose_Match_Count_OneOff'))` | 20 Aug capture |

## SetVariable actions — page creation / update branch

| Action | Value | Source |
|---|---|---|
| `Set_varPageAction_Created` | `Created` (literal) | 20 Aug capture |
| `Set_varOutputPageSelfUrl_Created` | `@outputs('Compose_PageSelfUrl_Created')` | 20 Aug capture |
| `Set_varOutputPageLink_Created` | `@outputs('Create_OneNote_Page')?['body']?['links']?['oneNoteWebUrl']?['href']` | 20 Aug capture |
| `Set_varPageAction_Created_OneOff` | `Created` (literal) | 20 Aug capture |
| `Set_varOutputPageSelfUrl_Created_OneOff` | `@outputs('Compose_PageSelfUrl_Created')` | 20 Aug capture |
| `Set_varOutputPageLink_Created_OneOff_Gate` | `@outputs('Create_OneNote_Page')?['body']?['links']?['oneNoteWebUrl']?['href']` | 20 Aug capture |
| `Set_varPageAction_ExistsNoCreate` | `Updated` (literal) | 20 Aug capture |
| `Set_varOutputPageSelfUrl_Existing` | `@variables('varFinalExistingPageSelfUrl')` | 20 Aug capture |
| `Set_varPageAction_UpdatedAppend` | `Updated` (literal) | 20 Aug capture |
| `Set_varOutputPageLink_Existing` | `@variables('varFinalExistingPageSelfUrl')` | 20 Aug capture |
| `Set_varOutputPageLink_Created_OneOff` | `@outputs('Create_Page_OneOff')?['body']?['links']?['oneNoteWebUrl']?['href']` | 20 Aug capture |
| `Set_varOutStatus` (final, before Respond) | `OK` (literal) | 20 Aug capture — still hardcoded, see open OutStatus gap |

## Key Compose actions (not variable-corruption-affected historically, but useful to have on hand)

| Action | Value | Notes |
|---|---|---|
| `Compose_Input_SeriesMasterId` | `@triggerBody()?['text_2']` | |
| `Compose_Input_MeetingTitle` | `@triggerBody()?['text_1']` | |
| `Compose_ExistingPageSelfUrl` | `@if(greater(length(body('Filter_Existing_Mapping')), 0), first(body('Filter_Existing_Mapping'))?['PageSelfUrl'], '')` | |
| `Compose_PageDecision` | `@if(not(empty(outputs('Compose_ExistingPageSelfUrl'))), 'PAGE_EXISTS', 'PAGE_NOT_FOUND')` | |
| `Compose_Match_Count` | `@length(body('Filter_Existing_Mapping'))` | |
| `Compose_SafeSectionName` | see full expression in `flow-reference-2026-08-20-full-peek-code-capture.md` — 43-char truncated, sanitised section name, prefixed `Mtg - ` | Long expression, worth copying verbatim from the dated capture rather than retyping |
| `Compose_SafePageTitle` | `@if(empty(trim(coalesce(triggerBody()?['text_1'], ''))), 'Untitled Meeting', substring(replace(replace(replace(replace(triggerBody()?['text_1'], '&', 'and'), '<', ''), '>', ''), '"', ''), 0, min(150, length(replace(replace(replace(replace(triggerBody()?['text_1'], '&', 'and'), '<', ''), '>', ''), '"', '')))))` | |
| `Compose_UpdateHtmlFragment` | **Hardcoded static string — see `bug-2026-08-20-update-fragment-discards-new-content.md`, this is a confirmed bug, not a value to blindly restore once fixed** | Do not restore this verbatim once the fix lands — check the bug doc first |
| `Compose_ExistingPageId` | `@last(split(variables('varOutputPageSelfUrl'), '/'))` | |
| `Compose_RealExistingPageId` | `@if(greater(length(outputs('Get_Pages_In_Section_Existing_Branch')?['body']?['value']), 0), first(outputs('Get_Pages_In_Section_Existing_Branch')?['body']?['value'])?['id'], '')` | This is the Bug 9 workaround — temporary by design, see design amendment doc before treating this as permanently correct |

## Key connector actions (parameters, not variables — less corruption-prone but included for completeness)

| Action | Key parameters |
|---|---|
| `Get items` (SharePoint) | `dataset: https://jsainsbury.sharepoint.com/sites/coplt`, `table: 186b3c9f-e758-4e85-83d5-685946614a0a` |
| `Create_OneNote_Page` | `notebookKey: Meeting Notes\|$\|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Meeting Notes`, `sectionId: @variables('varTargetSectionPagesUrl')`, `pageContent: <p>@{triggerBody()?['text_3']}</p>` |
| `Create_Page_OneOff` | Same connector/notebook/sectionId source as above — this is the Bug 5 failure point when `varTargetSectionPagesUrl` is blank |
| `Send_an_HTTP_request_to_SharePoint` (mapping write) | `parameters/uri: _api/web/lists/GetByTitle('RecurringMeetingSectionMap')/items`, method POST, body includes `Title, SeriesMasterId, MeetingTitle, SectionPagesUrl, Status: Active` |

---

## How to use this during a corruption incident

1. Confirm which actions actually lost their value (Peek Code each suspect action, or use Flow Checker to spot blank required fields — note Flow Checker has been shown to miss some blanked values, so don't rely on it alone).
2. Cross-check this table (or the fuller dated capture doc if this table is stale) for the correct expression.
3. Paste back exactly — do not retype from memory, since several of these expressions are long and a small typo (a missing `?`, wrong quote type) can reintroduce a different bug.
4. Save as draft, run Flow Checker, then **explicitly Publish** before testing — draft-only saves have caused false-negative retests before (see `handover-2026-08-16-bug9-closed-workaround-confirmed.md`).
5. Update this doc's "Last verified" date if anything here needed correcting during recovery.

---
*Compiled 20 August 2026 from the same container-level Peek Code capture as `flow-reference-2026-08-20-full-peek-code-capture.md`. Keep both in sync going forward — update this table whenever an expression is deliberately changed, not just after corruption recovery.*
