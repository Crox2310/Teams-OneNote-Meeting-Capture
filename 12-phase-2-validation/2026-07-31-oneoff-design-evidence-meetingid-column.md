# One-Off Existing-Page Resolution — Design Evidence, 31 July 2026

## Purpose

This doc collects the targeted evidence gathered on 31 July to answer the open design question from `handover-2026-07-30-oneoff-badrequest-live-trace-confirmation.md`: **how should the flow resolve "the existing page" for a one-off (non-recurring) meeting on recapture, given there's no `SeriesMasterId` to key on?**

Evidence gathering is now complete, including full raw JSON for every relevant condition block. This doc ends with a concrete, buildable design — see "Build plan" at the bottom. **No fix has been implemented in this session** — this is design only, ready for a future build session.

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
Confirms: the only match key is `SeriesMasterId`. One-off meetings have no `SeriesMasterId`, so this filter can never match a one-off meeting's row.

## Confirmed: Flow B's own trigger already labels `text_4` as `MeetingId` — not just "CalendarEventId"

```json
"text_4": { "title": "MeetingId", "type": "string", ... }
```
Whoever built Flow B already conceived of this value (the Outlook calendar event's own `id`, sourced from Flow A's `FA08 Get calendar view of events`) as *being* the meeting identifier. Wiring it into the mapping list's `MeetingId` column completes plumbing that was already named for this exact purpose.

**`MeetingId`'s intended purpose — confirmed by David:** scoped for a planned future feature (finding post-meeting transcriptions/recordings/actions and posting them into the correct OneNote page). Compatible with reusing it here. **One flagged, unresolved risk:** Teams transcript/recording APIs sometimes key off a Teams online-meeting ID (from `joinWebUrl`) rather than the plain Outlook calendar event ID used here — worth a check once the transcription feature is actually scoped. Not a blocker for today's fix.

## Confirmed: the complete execution chain (via `runAfter`, not assumed from list order)

Raw JSON for every relevant condition confirms this sequence, all within the same top-level path:

**`Condition_IsRecurring`** (`runAfter: varPageAction`) →
**`Condition_Mapping_Exists`** (`runAfter: Condition_IsRecurring`) →
**`Condition_Should_Create_Page`** (`runAfter: Condition_Mapping_Exists`) →
**`Condition_Is_Genuine_Existing_Page`** (nested inside `Condition_Should_Create_Page`'s `else`)

This corrects an earlier assumption in this investigation (and in the 30 July doc) that these conditions might run somewhat independently based on their order in the "Go to Operation" list — that list's display order does **not** reflect true execution order. The chain above is confirmed directly from each action's own `runAfter` property.

### `Condition_IsRecurring` — full structure

True (recurring) branch: `Compose_Input_SeriesMasterId` → `Compose_Input_MeetingTitle` → `Filter_Existing_Mapping` (keyed on `SeriesMasterId`) → `Compose_ExistingPageSelfUrl` → **`Compose_PageDecision`** (`PAGE_EXISTS` / `PAGE_NOT_FOUND`) → `Compose_Match_Count` → sets `varFinalExistingPageSelfUrl` / `varFinalPageDecision` / `varFinalMatchCount`.

False (one-off) branch: builds the safe section title → `Get_Sections_OneOff` → `Filter_OneNote_Section_OneOff` → `Compose_Section_Match_Count_OneOff` → `Condition_Section_Exists_OneOff` (finds or creates the section, sets `varTargetSectionPagesUrl` and `varOneNoteResolverResult`). **Nothing in this branch sets `Compose_PageDecision`, `Compose_ExistingPageSelfUrl`, `Compose_Match_Count`, or any of the three `varFinal*` variables.**

### `Condition_Mapping_Exists` — full structure

Branches on `variables('varFinalMatchCount') > 0`.

True branch: sets `varTargetSectionPagesUrl` and `varOneNoteResolverResult = "ExistingMapping"` from the (recurring-only) `Filter_Existing_Mapping` result.

False branch (`runAfter: Condition_Section_Exists_Recurring`, i.e. this branch itself contains a **second, full copy of the recurring section-resolution logic** — `Get_Sections_Recurring`, `Filter_OneNote_Section_Recurring`, `Condition_Section_Exists_Recurring`, `Create_Section_Recurring`) plus, gated by `Condition_Should_Write_Mapping`, the mapping-table write (`Send_an_HTTP_request_to_SharePoint`).

**Because `varFinalMatchCount` is never set on the one-off path, it evaluates empty → coerced to `'0'` → this condition's True branch is structurally unreachable for one-off meetings; every one-off meeting takes this False branch unconditionally.** This means the recurring-style section-resolution logic here runs a second, redundant time on every one-off capture (after the one-off-specific version already ran inside `Condition_IsRecurring`). Appears harmless in practice — same section name, so it just re-finds what was just resolved — and the SharePoint write is correctly gated on `IsRecurring == 'true'` so it doesn't write bad data. But it is wasted work on every single one-off capture. **Worth its own backlog item, separate from this fix.**

### `Condition_Should_Create_Page` — the critical finding

Branches on:
```json
"@equals(outputs('Compose_PageDecision'), 'PAGE_NOT_FOUND')"
```

**`Compose_PageDecision` is only ever defined inside `Condition_IsRecurring`'s True (recurring) branch.** For every one-off meeting, this expression references the output of an action that never executed.

This matches exactly what was observed live on 30 July: the flow did not error at this check — it silently evaluated false and proceeded into the "page exists, don't create" (`else`) branch, which is where the `BadRequest` chain traced on 30 July originated. This is strong evidence that `outputs()` of a never-executed sibling-branch action resolves quietly to null here rather than throwing.

**The consequence is broader than previously understood: this isn't only a recapture bug. It means every one-off meeting — including a brand-new one-off meeting with no existing page at all — incorrectly takes the "reuse existing page" branch instead of "create a new page," because the decision variable that should say `PAGE_NOT_FOUND` is simply never computed for the one-off path.** The underlying create-vs-reuse decision is undefined, not just "missing a lookup for recapture."

A second instance of the same category of bug: inside `Condition_Is_Genuine_Existing_Page`'s True branch, `Set_varOutputPageLink_Existing` reads `outputs('Compose_ExistingPageSelfUrl')` — also only ever defined in the recurring branch. Same root problem, different symptom (a broken output link rather than a failed write).

### Discrepancy flagged, not resolved: `Set_varOutputPageSelfUrl_Existing`'s current `value`

Today's raw JSON capture shows:
```json
"Set_varOutputPageSelfUrl_Existing": {
  "type": "SetVariable",
  "inputs": { "name": "varOutputPageSelfUrl", "value": "\"\"" }
}
```
i.e. a `value` key **is present**, containing the two-character string `""` (two literal quote characters) — not a true empty string. This differs from the 30 July capture, which showed the `value` key **missing entirely** from this same action. Not yet reconciled — possibly the flow was edited between the two captures, possibly a copy/paste escaping artifact in one of the two captures. **Needs a direct Peek Code confirmation before the build session**, since it changes exactly what needs editing (add a `value` key vs. correct an existing one) — low functional impact either way, since both produce an unusable `pageId` downstream, but worth getting right.

## Confirmed: the full recurring-path mapping-write lifecycle

**Write 1 — `Send an HTTP request to SharePoint`** (inside `Condition_Should_Write_Mapping` → `True`, itself inside `Condition_Mapping_Exists`'s False branch):
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
One action correctly handles both "this meeting already had a row" and "we just created its row," via an inline `if()` choosing which ID to target.

## Confirmed: the one-off path has zero SharePoint interaction, anywhere

`Set varOneNoteResolverResult Created OneOff`, `Set varTargetSectionPagesUrl OneOff Created`, and `Create Page OneOff` are all confirmed free of any SharePoint reference — pure OneNote/in-memory logic. Combined with the `Condition_Should_Create_Page` finding above, the full picture is now clear: one-off meetings have no mapping-table memory *and* their create-vs-reuse decision is undefined, not just defaulting to the wrong thing.

## Build plan — revised and ready for implementation

The fix needs to make the one-off branch populate the same decision variables the recurring branch already does, not just add a filter. Five actions, each templated directly on a recurring-path equivalent, all placed inside `Condition_IsRecurring`'s **False (one-off)** branch:

**1. `Filter_Existing_Mapping_OneOff`** — mirrors `Filter_Existing_Mapping`, keyed on `MeetingId` instead of `SeriesMasterId`:
```json
{
  "type": "Query",
  "inputs": {
    "from": "@body('Get_items')?['value']",
    "where": "@equals(item()?['MeetingId'],triggerBody()?['text_4'])"
  }
}
```

**2. `Compose_ExistingPageSelfUrl_OneOff`** — mirrors `Compose_ExistingPageSelfUrl`:
```json
"@if(greater(length(body('Filter_Existing_Mapping_OneOff')), 0), first(body('Filter_Existing_Mapping_OneOff'))?['PageSelfUrl'], '')"
```

**3. `Compose_PageDecision_OneOff`** — mirrors `Compose_PageDecision`. **This is the critical addition** — without this, `Condition_Should_Create_Page` remains broken for one-off meetings regardless of anything else built:
```json
"@if(not(empty(outputs('Compose_ExistingPageSelfUrl_OneOff'))), 'PAGE_EXISTS', 'PAGE_NOT_FOUND')"
```

**4. Three `SetVariable` actions mirroring `varFinalExistingPageSelfUrl_1` / `varFinalPageDecision_1` / `varFinalMatchCount_1`**, sourcing from the OneOff Compose actions above instead of the recurring ones. These feed `Condition_Should_Create_Page` (via `Compose_PageDecision`, so a decision must be **read from a single shared point** — see open question below) and downstream link-building.

**5. Fix `Set_varOutputPageSelfUrl_Existing`'s `value`** (exact current state pending the discrepancy check above) to reference the correct final variable, matching however the recurring path does it.

**6. One-off equivalents of the two-write lifecycle** (`Send an HTTP request to SharePoint OneOff`, `HTTP Update SP PageSelfUrl OneOff`), keyed on `MeetingId`, so a *second* capture of the same one-off meeting has something for step 1 to find. Same `if()`-based dual-purpose ID-targeting pattern as the recurring `HTTP Update SP PageSelfUrl`.

### Open question this raises, not yet resolved: does `Condition_Should_Create_Page` need editing too?

`Condition_Should_Create_Page` currently reads `outputs('Compose_PageDecision')` — the *recurring* Compose action specifically, by name. Simply adding a parallel `Compose_PageDecision_OneOff` in the one-off branch does **not** make `Condition_Should_Create_Page` see it — Power Automate `outputs()` references a specific named action, not "whichever ran." Two options:
- **(a)** Change `Condition_Should_Create_Page`'s expression to check a variable instead of an action output directly — e.g. set a shared `varPageDecision` variable at the end of both `Condition_IsRecurring` branches, then have `Condition_Should_Create_Page` check `variables('varPageDecision')` instead of `outputs('Compose_PageDecision')`. Cleaner, but touches an existing, currently-working recurring-path expression — needs care and a recurring-path regression test, not just a one-off test.
- **(b)** Duplicate `Condition_Should_Create_Page`'s logic separately for the one-off branch (more duplication, but zero risk to the existing recurring path).

**This needs to be decided before building, not discovered mid-build.** (a) is architecturally cleaner and matches how `varFinalPageDecision` already exists as a variable (currently unused downstream, per the trace above — suggesting it may have been *intended* for exactly this purpose already and never wired up, similar to the `MeetingId` column finding). Worth checking whether `variables('varFinalPageDecision')` (already set by both a recurring `varFinalPageDecision_1` action, once one-off gets its equivalent) could simply replace `outputs('Compose_PageDecision')` in `Condition_Should_Create_Page`'s expression — likely the minimal, lowest-risk version of option (a).

## Before building — final checklist

1. ~~Confirm `text_4` is the meeting identifier on Flow B's own trigger~~ — **DONE.** Confirmed titled `MeetingId` directly.
2. ~~Confirm exact execution order/placement via canvas~~ — **DONE, and superseded by something better** — full raw JSON for all four conditions now captured, giving exact `runAfter` chains rather than inferred canvas positions.
3. **Resolve the `Set_varOutputPageSelfUrl_Existing` value-field discrepancy** (missing key vs. literal `""` string) via direct Peek Code check.
4. **Decide how `Condition_Should_Create_Page` will see the one-off decision** — option (a) vs (b) above. Recommend checking whether `varFinalPageDecision` can simply replace the current `outputs('Compose_PageDecision')` reference.
5. **Re-check the `Status` column rendering discrepancy** — low priority.
6. Once built: live-test recapturing the same one-off meeting twice, **and** run a recurring-meeting regression test if `Condition_Should_Create_Page`'s expression itself gets touched (option a) — same evidence-first standard as 30 July, Peek Code + raw run output, not "looks right."

## Status

**Design substantially more complete and more correct than the previous revision of this doc** — the earlier build plan would not have fully fixed the bug, since it didn't address `Condition_Should_Create_Page`'s broken decision input. Not yet implemented. One real open decision remains (item 4 above) before this is fully buildable without ambiguity.
