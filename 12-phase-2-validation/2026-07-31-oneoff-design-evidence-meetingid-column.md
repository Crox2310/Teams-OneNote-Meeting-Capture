# One-Off Existing-Page Resolution — Design, 31 July 2026 (FINAL — Option A decided)

## Purpose

This doc collects the evidence and final design to fix the one-off existing-page gap identified in `handover-2026-07-30-oneoff-badrequest-live-trace-confirmation.md`. **Design is now finalised.** David has decided on Option A (below) for the one open architectural question. **No fix has been implemented in this session** — this is the complete design, ready for a build session.

## Confirmed: Flow B's mapping lookup is keyed only on `SeriesMasterId`

**`Get items`** (SharePoint `GetItems`, dataset `https://jsainsbury.sharepoint.com/sites/coplt`, table `186b3c9f-e758-4e85-83d5-685946614a0a`) — pulls the entire mapping list, no server-side filter.

**`Filter Existing Mapping`**:
```json
{
  "type": "Query",
  "inputs": {
    "from": "@body('Get_items')?['value']",
    "where": "@equals(item()?['SeriesMasterId'],triggerBody()?['text_2'])"
  }
}
```
One-off meetings have no `SeriesMasterId`, so this can never match a one-off meeting's row.

## Confirmed: Flow B's own trigger already labels `text_4` as `MeetingId`

```json
"text_4": { "title": "MeetingId", "type": "string", ... }
```
Sourced from Flow A's `FA08 Get calendar view of events` (the Outlook event's own `id`), returned to the Topic as `Topic.CalendarEventId`, passed into Flow B where the trigger itself already calls it `MeetingId`. This value exists and is non-blank for every meeting type once selected.

**`MeetingId`'s intended purpose — confirmed by David:** scoped for a planned future feature (finding post-meeting transcriptions/recordings/actions and posting them into the correct OneNote page). **One flagged, unresolved risk, not a blocker today:** Teams transcript/recording APIs sometimes key off a Teams online-meeting ID (from `joinWebUrl`) rather than the plain Outlook calendar event ID used here — worth checking once the transcription feature is actually scoped.

## Confirmed: the complete execution chain (via `runAfter`, not assumed from list order)

**`Condition_IsRecurring`** (`runAfter: varPageAction`) →
**`Condition_Mapping_Exists`** (`runAfter: Condition_IsRecurring`) →
**`Condition_Should_Create_Page`** (`runAfter: Condition_Mapping_Exists`) →
**`Condition_Is_Genuine_Existing_Page`** (nested inside `Condition_Should_Create_Page`'s `else`)

This corrects an earlier assumption (in this doc and the 30 July doc) that these conditions might run independently, based on their order in the "Go to Operation" list — that list's display order does **not** reflect true execution order.

### `Condition_IsRecurring` — full structure

True (recurring) branch: `Compose_Input_SeriesMasterId` → `Compose_Input_MeetingTitle` → `Filter_Existing_Mapping` → `Compose_ExistingPageSelfUrl` → `Compose_PageDecision` (`PAGE_EXISTS`/`PAGE_NOT_FOUND`) → `Compose_Match_Count` → sets `varFinalExistingPageSelfUrl` / `varFinalPageDecision` / `varFinalMatchCount`.

False (one-off) branch: builds the safe section title → `Get_Sections_OneOff` → `Filter_OneNote_Section_OneOff` → `Compose_Section_Match_Count_OneOff` → `Condition_Section_Exists_OneOff` (finds/creates section, sets `varTargetSectionPagesUrl`/`varOneNoteResolverResult`). **Nothing here sets `Compose_PageDecision`, `Compose_ExistingPageSelfUrl`, `Compose_Match_Count`, or the three `varFinal*` variables.**

### `Condition_Mapping_Exists` — full structure

Branches on `variables('varFinalMatchCount') > 0`. True branch: sets `varTargetSectionPagesUrl`/`varOneNoteResolverResult = "ExistingMapping"` from `Filter_Existing_Mapping`. False branch contains a second, redundant copy of the recurring section-resolution logic, plus (gated by `Condition_Should_Write_Mapping`) the mapping-table write.

Since `varFinalMatchCount` is never set on the one-off path, this condition's True branch is structurally unreachable for one-off meetings — every one-off meeting takes the False branch unconditionally, redundantly re-running recurring-style section resolution. Appears harmless (same section name, re-finds what was just resolved; the SharePoint write is correctly gated on `IsRecurring == 'true'`), but is wasted work on every one-off capture. **Separate backlog item, not part of this fix.**

### `Condition_Should_Create_Page` — the critical finding

Branches on `@equals(outputs('Compose_PageDecision'), 'PAGE_NOT_FOUND')`. Since `Compose_PageDecision` only exists in the recurring branch, this references a never-executed action for every one-off meeting. Matches exactly what was observed live on 30 July: no error, silent fall-through to the "page exists, don't create" branch — where the traced `BadRequest` originated.

**This means every one-off meeting — including a brand-new one with no existing page — incorrectly takes "reuse existing page" instead of "create new page."** Not just a recapture bug; the underlying decision is undefined for the whole one-off path.

Second instance of the same bug category: `Set_varOutputPageLink_Existing` (inside `Condition_Is_Genuine_Existing_Page`'s True branch) reads `outputs('Compose_ExistingPageSelfUrl')` — also recurring-only.

### Resolved: `Set_varOutputPageSelfUrl_Existing`'s current `value`

Confirmed via direct screenshot: the `value` key **is present**, set to the literal two-character string `"\"\""` (two quote characters, not a true empty string):
```json
"Set_varOutputPageSelfUrl_Existing": {
  "type": "SetVariable",
  "inputs": { "name": "varOutputPageSelfUrl", "value": "\"\"" }
}
```
This is what's currently live — the earlier 30 July capture showing a missing `value` key entirely appears to have been a slightly different/earlier state. Not functionally significant either way (both produce an unusable downstream value) — now resolved, no longer an open question.

## Confirmed: the full recurring-path mapping-write lifecycle

**Write 1 — `Send an HTTP request to SharePoint`** (inside `Condition_Should_Write_Mapping` → `True`, inside `Condition_Mapping_Exists`'s False branch):
```json
{
  "parameters/method": "POST",
  "parameters/uri": "_api/web/lists/GetByTitle('RecurringMeetingSectionMap')/items",
  "parameters/body": "{ \"Title\": \"Mapping\", \"SeriesMasterId\": \"@{outputs('Compose_Input_SeriesMasterId')}\", \"MeetingTitle\": \"@{outputs('Compose_Input_MeetingTitle')}\", \"SectionPagesUrl\": \"@{variables('varTargetSectionPagesUrl')}\", \"Status\": \"Active\" }"
}
```

**Write 2 — `HTTP Update SP PageSelfUrl`** (`runAfter: Compose_PageSelfUrl_Created`, inside `Condition_Should_Create_Page`'s True branch):
```json
{
  "parameters/method": "POST",
  "parameters/uri": "_api/web/lists/GetByTitle('RecurringMeetingSectionMap')/items(@{if(greater(length(body('Filter_Existing_Mapping')),0), first(body('Filter_Existing_Mapping'))?['ID'], body('Send_an_HTTP_request_to_SharePoint')?['ID'])})",
  "parameters/headers": { "IF-MATCH": "*", "X-HTTP-Method": "MERGE" },
  "parameters/body": "{ \"PageSelfUrl\": \"@{outputs('Compose_PageSelfUrl_Created')}\" }"
}
```
One action handles both "existing row" and "just-created row" via an inline `if()`.

## Confirmed: the one-off path has zero SharePoint interaction, anywhere

`Set varOneNoteResolverResult Created OneOff`, `Set varTargetSectionPagesUrl OneOff Created`, `Create Page OneOff` — all confirmed free of SharePoint references.

## DECISION: Option A — read the page decision from a shared variable, not an action-specific output

David has chosen Option A: change `Condition_Should_Create_Page` and `Set_varOutputPageLink_Existing` to read from **variables** (`varFinalPageDecision`, `varFinalExistingPageSelfUrl`) rather than referencing the recurring branch's Compose actions by name directly. Both the recurring and (new) one-off branches will write into the same shared variables; the downstream conditions become meeting-type-agnostic.

This is the cleaner, more durable fix, and it happens to repair **both** instances of the "reads a recurring-only output" bug in one change, not just the one that caused the 30 July failure. Trade-off, explicitly accepted: this touches two expressions that currently work correctly for recurring meetings, so a **recurring-path regression test is required** alongside the one-off test before considering this done.

## Final build plan

**All new actions below sit inside `Condition_IsRecurring`'s False (one-off) branch**, mirroring the recurring branch's shape exactly:

**1. `Filter_Existing_Mapping_OneOff`**:
```json
{ "type": "Query", "inputs": { "from": "@body('Get_items')?['value']", "where": "@equals(item()?['MeetingId'],triggerBody()?['text_4'])" } }
```

**2. `Compose_ExistingPageSelfUrl_OneOff`**:
```json
"@if(greater(length(body('Filter_Existing_Mapping_OneOff')), 0), first(body('Filter_Existing_Mapping_OneOff'))?['PageSelfUrl'], '')"
```

**3. `Compose_PageDecision_OneOff`**:
```json
"@if(not(empty(outputs('Compose_ExistingPageSelfUrl_OneOff'))), 'PAGE_EXISTS', 'PAGE_NOT_FOUND')"
```

**4. `Compose_Match_Count_OneOff`** (optional but recommended for consistency/future transcription feature use):
```json
"@length(body('Filter_Existing_Mapping_OneOff'))"
```

**5. Three `SetVariable` actions**, one-off branch versions of `varFinalExistingPageSelfUrl_1` / `varFinalPageDecision_1` / `varFinalMatchCount_1`, sourcing from the four OneOff Compose actions above instead of the recurring ones. **Same variable names** (`varFinalExistingPageSelfUrl`, `varFinalPageDecision`, `varFinalMatchCount`) — this is what makes Option A work: both branches write to the same variables.

**6. Edit `Condition_Should_Create_Page`'s expression**:
- From: `"@equals(outputs('Compose_PageDecision'), 'PAGE_NOT_FOUND')"`
- To: `"@equals(variables('varFinalPageDecision'), 'PAGE_NOT_FOUND')"`

**7. Edit `Set_varOutputPageLink_Existing`'s value**:
- From: `"@outputs('Compose_ExistingPageSelfUrl')"`
- To: `"@variables('varFinalExistingPageSelfUrl')"`

**8. Fix `Set_varOutputPageSelfUrl_Existing`'s `value`** — from the literal `"\"\""` placeholder to:
```json
"@variables('varFinalExistingPageSelfUrl')"
```

**9. One-off equivalents of the two-write mapping lifecycle** — `Send an HTTP request to SharePoint OneOff` and `HTTP Update SP PageSelfUrl OneOff`, keyed on `MeetingId` (recurring uses `SeriesMasterId`), same `if()`-based dual-purpose ID-targeting pattern as the recurring `HTTP Update SP PageSelfUrl`. Needed so a *second* capture of the same one-off meeting has a row for step 1 to find.

## Before building — final checklist

1. ~~Confirm `text_4` is the meeting identifier on Flow B's own trigger~~ — **DONE.**
2. ~~Confirm exact execution order~~ — **DONE**, full raw JSON captured for all four conditions.
3. ~~Resolve the `Set_varOutputPageSelfUrl_Existing` value-field discrepancy~~ — **DONE**, confirmed live via screenshot.
4. ~~Decide how `Condition_Should_Create_Page` sees the one-off decision~~ — **DONE, Option A.**
5. **Re-check the `Status` column rendering discrepancy** — low priority, cheap to resolve.
6. **Build session:** implement points 1–9 of the build plan, in order.
7. **Test session:** live-test recapturing the same one-off meeting twice (Peek Code + raw run output at each new action, per the 30 July evidence-first standard) **and** a recurring-meeting regression test (since steps 6–7 touch previously-working recurring-path expressions).

## Status

**Design finalised.** All open questions resolved. Ready for a dedicated build session. Recommend treating checklist items 6–7 as that session's scope, in that order — build fully, then test both paths, rather than testing incrementally mid-build.
