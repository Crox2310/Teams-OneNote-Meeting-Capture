# One-Off Existing-Page Resolution — Design Evidence, 31 July 2026

## Purpose

This doc collects the targeted evidence gathered on 31 July to answer the open design question from `handover-2026-07-30-oneoff-badrequest-live-trace-confirmation.md`: **how should the flow resolve "the existing page" for a one-off (non-recurring) meeting on recapture, given there's no `SeriesMasterId` to key on?**

Evidence gathering is now complete. This doc ends with a concrete, buildable design — see "Build plan" at the bottom. **No fix has been implemented in this session** — this is design only, ready for a future build session.

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
Confirms: the only match key is `SeriesMasterId`. One-off meetings have no `SeriesMasterId`, so this filter can never match a one-off meeting's row — consistent with everything found on 30 July.

## Confirmed: `CalendarEventId` is a stable, always-available identifier, computed in Flow A

Traced `OutCalendarEventId` through Flow A's candidate-resolution branches:

- **Single/selected match:** `FA21 Compose OutCalendarEventId`: `@coalesce(outputs('FA19_Compose_SelectedEvent')?['id'], '')`
- **Multi-candidate, one selected:** `FA30 Compose OutCalendarEventId`: `@coalesce(outputs('FA28_Compose_SingleEvent')?['id'], '')`
- **No match:** `FA27F Compose OutCalendarEventId NoMatch`: `@string('')`
- **Multi-candidate, not yet selected:** `FA42 Compose OutCalendarEventId Multi`: `@string('')`

Source: **`FA08 Get calendar view of events`** (Office 365 Outlook `GetEventsCalendarViewV3`). In every branch where a specific meeting is resolved, `OutCalendarEventId` is that meeting's own Outlook event `id`.

**Non-blank for every meeting type — recurring or one-off — once selected.** Already returned by Flow A to the Topic (`calendareventid: Topic.CalendarEventId`) and already passed by the Topic into Flow B (`text_4: =Topic.CalendarEventId`). No new plumbing needed to get this value to Flow B — it already arrives at the trigger.

## Confirmed: the SharePoint mapping list already has an unused `MeetingId` column

Live `RecurringMeetingSectionMap` list (`jsainsbury.sharepoint.com/sites/coplt`, list `MeetingNoteIndex`). Full column set, from the one existing row ("Mapping"):

| Column | Populated on existing row? |
|---|---|
| Title | Yes |
| SeriesMasterId | Yes |
| MeetingTitle | Yes |
| SectionPagesUrl | Yes |
| PageSelfUrl | Yes |
| LastUsedAtUtc | Empty |
| Status | Empty (rendering discrepancy noted, unresolved, low priority) |
| SectionSelfUrl | Empty |
| LastUpdatedUtc | Empty |
| SectionId | Empty |
| **MeetingId** | **Empty** |

Nothing in Flow B currently reads or writes `MeetingId`, `SectionId`, `SectionSelfUrl`, or `LastUpdatedUtc`.

**`MeetingId`'s intended purpose — confirmed by David:** scoped for a planned future feature (finding post-meeting transcriptions/recordings/actions and posting them to the right OneNote page). Compatible with reusing it for one-off page resolution — both need a stable per-meeting identifier. **One flagged risk, not yet resolved:** Teams transcript/recording APIs sometimes key off a Teams online-meeting ID (from `joinWebUrl`) rather than the plain Outlook calendar event ID used here — worth a deliberate check once the transcription feature is actually scoped, in case `MeetingId` ends up needing to hold a different value for that purpose. Not a blocker for today's fix.

## Confirmed: the full recurring-path mapping-write lifecycle

**Write 1 — `Send an HTTP request to SharePoint`** (fires once, during section resolution, inside `Condition Should Write Mapping` → `True`, under `Condition Section Exists Recurring`):
```json
{
  "parameters/method": "POST",
  "parameters/uri": "_api/web/lists/GetByTitle('RecurringMeetingSectionMap')/items",
  "parameters/body": "{ \"Title\": \"Mapping\", \"SeriesMasterId\": \"@{outputs('Compose_Input_SeriesMasterId')}\", \"MeetingTitle\": \"@{outputs('Compose_Input_MeetingTitle')}\", \"SectionPagesUrl\": \"@{variables('varTargetSectionPagesUrl')}\", \"Status\": \"Active\" }"
}
```
Creates the row. No `MeetingId`, no `PageSelfUrl` yet (page doesn't exist at this point).

**Write 2 — `HTTP Update SP PageSelfUrl`** (fires after the page is created, `runAfter: Compose_PageSelfUrl_Created`):
```json
{
  "parameters/method": "POST",
  "parameters/uri": "_api/web/lists/GetByTitle('RecurringMeetingSectionMap')/items(@{if(greater(length(body('Filter_Existing_Mapping')),0), first(body('Filter_Existing_Mapping'))?['ID'], body('Send_an_HTTP_request_to_SharePoint')?['ID'])})",
  "parameters/headers": { "IF-MATCH": "*", "X-HTTP-Method": "MERGE" },
  "parameters/body": "{ \"PageSelfUrl\": \"@{outputs('Compose_PageSelfUrl_Created')}\" }"
}
```
Updates that same row's `PageSelfUrl`. The target item ID is chosen with an inline `if()`: use `Filter_Existing_Mapping`'s match if one existed, otherwise use the ID just returned by Write 1's POST. **One action correctly handles both "this meeting already had a row" and "we just created its row" cases.**

## Confirmed: the one-off path has zero SharePoint interaction, anywhere

**`Set varOneNoteResolverResult Created OneOff`**: `{"value": "CreatedSection"}` — status flag only.
**`Set varTargetSectionPagesUrl OneOff Created`**: `{"value": "@outputs('Create_Section_OneOff')?['body']?['pagesUrl']"}` — purely in-memory.
**`Create Page OneOff`**: plain OneNote `CreatePageInSection` call — no SharePoint write.

The one-off path creates sections and pages in complete isolation from the mapping list. It has no memory of anything it does.

## Build plan — ready for implementation

Four new/modified actions, each directly templated on a recurring-path equivalent:

**1. Modify `Send an HTTP request to SharePoint`** (recurring write) — add `MeetingId` to the body:
```json
"MeetingId": "@{triggerBody()?['text_4']}"
```
(assuming `text_4` is still the `CalendarEventId` trigger parameter — confirm against Flow B's `When an agent calls the flow` schema before building, since parameter numbering could differ from Flow A's/Topic's naming).

**2. New action: `Filter Existing Mapping OneOff`** — same shape as `Filter Existing Mapping`, on the one-off path:
```json
{
  "type": "Query",
  "inputs": {
    "from": "@body('Get_items')?['value']",
    "where": "@equals(item()?['MeetingId'],triggerBody()?['text_4'])"
  }
}
```
Placement: needs to run early enough that its result can feed both page-decision logic and, later, `varOutputPageSelfUrl Existing` — likely mirroring where `Filter Existing Mapping` sits relative to `Condition IsRecurring`, i.e. probably needs to sit in the `Condition IsRecurring` → `False` (one-off) branch, in roughly the position `Filter Existing Mapping` occupies on the `True` (recurring) side.

**3. Fix `Set varOutputPageSelfUrl Existing`'s blank `value` field** — the original Pattern 6 defect from 30 July. Once (2) exists, the correct expression is analogous to the recurring path's `varFinalExistingPageSelfUrl` population:
```json
"value": "@first(body('Filter_Existing_Mapping_OneOff'))?['PageSelfUrl']"
```
(Exact source action name depends on what gets built in step 2 — adjust to match.)

**4. New actions: one-off equivalents of the two-write lifecycle:**
- **`Send an HTTP request to SharePoint OneOff`** — same shape as the recurring write, but with `MeetingId` as the primary key instead of `SeriesMasterId` (which one-off meetings don't have — likely omit or leave blank), gated by an equivalent of `Condition Should Write Mapping` for the one-off path (i.e. only write if `Filter Existing Mapping OneOff` found nothing).
- **`HTTP Update SP PageSelfUrl OneOff`** — same `if()`-based item-targeting pattern as the recurring `HTTP Update SP PageSelfUrl`, `runAfter` the one-off page-self-URL Compose action, referencing `Filter Existing Mapping OneOff` / the one-off create-write's returned ID instead of the recurring equivalents.

**Suggested placement:** mirror the recurring path's structure exactly — Write (4a) goes near `Condition Section Exists OneOff` (parallel to where the recurring write sits near `Condition Section Exists Recurring`); Update (4b) goes after `Create Page OneOff` / its Compose-PageSelfUrl step, parallel to where `HTTP Update SP PageSelfUrl` sits after `Create OneNote Page`.

## Before building — final checks

1. **Confirm `text_4` is really `CalendarEventId` on Flow B's own trigger schema** (not just assumed from the Topic YAML's binding) — open `When an agent calls the flow`'s Code view and check the `text_4` schema title directly.
2. **Confirm the exact action names to branch off** by viewing `Condition IsRecurring`'s `False` branch and `Condition Section Exists OneOff`'s structure directly, to place the four new actions in exactly the right spot rather than guessing from the pattern alone.
3. **Re-check the `Status` column discrepancy** noted above — low priority, but cheap to resolve while already in the list.
4. Once built, needs a live test recapturing the same one-off meeting twice, checking Peek Code + raw run output at each new action — same evidence-first standard as the 30 July investigation, not just "it looks right."

## Status

**Design complete and buildable.** Not yet implemented. Recommend treating the "before building" checks above as the first 15 minutes of the next build session, then implementing points 1–4 of the build plan in order, then live-testing per point 4 of "before building."
