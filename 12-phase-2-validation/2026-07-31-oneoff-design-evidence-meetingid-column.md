# One-Off Existing-Page Resolution — Design Evidence, 31 July 2026

## Purpose

This doc collects the targeted evidence gathered on 31 July to answer the open design question from `handover-2026-07-30-oneoff-badrequest-live-trace-confirmation.md`: **how should the flow resolve "the existing page" for a one-off (non-recurring) meeting on recapture, given there's no `SeriesMasterId` to key on?**

This is evidence-gathering only — no fix implemented, no design finalised yet. Material was gathered from full Flow A and Flow B screenshot exports (Connectors, Go to Operation, High E2E View) plus a direct look at the live SharePoint mapping list.

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
Confirms: the only match key is `SeriesMasterId`. One-off meetings have no `SeriesMasterId`, so this filter can never match a one-off meeting's row (if one existed) — consistent with everything found on 30 July.

## Confirmed: `CalendarEventId` is a stable, always-available identifier, computed in Flow A

Traced `OutCalendarEventId` through Flow A's candidate-resolution branches:

- **Single/selected match:** `FA21 Compose OutCalendarEventId`: `@coalesce(outputs('FA19_Compose_SelectedEvent')?['id'], '')`
- **Multi-candidate, one selected:** `FA30 Compose OutCalendarEventId`: `@coalesce(outputs('FA28_Compose_SingleEvent')?['id'], '')`
- **No match:** `FA27F Compose OutCalendarEventId NoMatch`: `@string('')` — explicitly blank when there's no meeting
- **Multi-candidate, not yet selected:** `FA42 Compose OutCalendarEventId Multi`: `@string('')` — also blank until a specific meeting is chosen

The underlying source is **`FA08 Get calendar view of events`** (Office 365 Outlook `GetEventsCalendarViewV3`), and in every branch where a specific meeting is actually resolved, `OutCalendarEventId` is that meeting's own Outlook event `id`.

**This value exists and is non-blank for every meeting type — recurring or one-off** — once a specific meeting has been selected. It is already returned by Flow A to the Topic (`calendareventid: Topic.CalendarEventId`, confirmed in the 31 July Topic YAML review) and already passed by the Topic into Flow B (`text_4: =Topic.CalendarEventId`, same review). No new plumbing is required to get this value to Flow B — it's already arriving at the trigger.

## New finding: the SharePoint mapping list already has an unused `MeetingId` column

Viewed the live `RecurringMeetingSectionMap` list directly (SharePoint site `jsainsbury.sharepoint.com/sites/coplt`, list `MeetingNoteIndex`). Full column set, assembled by scrolling across the one existing row ("Mapping"):

| Column | Populated on existing row? |
|---|---|
| Title | Yes ("Mapping") |
| SeriesMasterId | Yes |
| MeetingTitle | Yes |
| SectionDisplayName | (not visible in captured view) |
| SectionPagesUrl | Yes |
| NotebookName | (not visible in captured view) |
| CreatedAtUtc | (not visible in captured view) |
| LastUsedAtUtc | **Empty** |
| Status | **Empty** (though a "Status" column with an "Active" badge rendering was visible — worth re-checking, see note below) |
| SectionSelfUrl | **Empty** |
| PageSelfUrl | Yes (this is the field `Compose_ExistingPageSelfUrl` reads) |
| LastUpdatedUtc | **Empty** |
| SectionId | **Empty** |
| **MeetingId** | **Empty** |

**Nothing in Flow B currently reads or writes `MeetingId`, `SectionId`, `SectionSelfUrl`, or `LastUpdatedUtc`.** These columns exist in the schema but are structurally unused by the flow as it currently stands.

**Note on `Status`:** one screenshot showed a "Status" column rendering an "Active" badge for the existing row, which appears to contradict "empty" — this needs re-confirming directly (possibly a SharePoint choice-column default rendering, or the column order shifted between screenshots during horizontal scroll). Flagging as unconfirmed rather than asserting either way.

## Confirmed: `Send an HTTP request to SharePoint` — the only mapping-list write in the flow

```json
{
  "type": "OpenApiConnection",
  "inputs": {
    "parameters": {
      "dataset": "https://jsainsbury.sharepoint.com/sites/coplt",
      "parameters/method": "POST",
      "parameters/uri": "_api/web/lists/GetByTitle('RecurringMeetingSectionMap')/items",
      "parameters/body": "{\n  \"Title\": \"Mapping\",\n  \"SeriesMasterId\": \"@{outputs('Compose_Input_SeriesMasterId')}\",\n  \"MeetingTitle\": \"@{outputs('Compose_Input_MeetingTitle')}\",\n  \"SectionPagesUrl\": \"@{variables('varTargetSectionPagesUrl')}\",\n  \"Status\": \"Active\"\n}"
    },
    "host": { "connection": "shared_sharepointonline", "operationId": "HttpRequest" }
  }
}
```

Only 5 fields written: `Title`, `SeriesMasterId`, `MeetingTitle`, `SectionPagesUrl`, `Status`. No `MeetingId`, no `PageSelfUrl` at this point (that's added later, presumably via a separate PATCH-style update once the page itself exists — not yet inspected in detail).

This action sits inside `Condition Should Write Mapping` → `True`, itself nested under `Condition Section Exists Recurring` — i.e. it only ever fires on the **recurring** path. Consistent with everything else found: one-off meetings never reach this action at all.

## Confirmed: the one-off path has zero SharePoint mapping-list interaction, anywhere

Three actions checked directly, closing out the last open question from earlier in the day:

**`Set varOneNoteResolverResult Created OneOff`**:
```json
{ "type": "SetVariable", "inputs": { "name": "varOneNoteResolverResult", "value": "CreatedSection" } }
```
Pure status-flag string, same shape as the recurring equivalent. No mapping/CalendarEventId reference.

**`Set varTargetSectionPagesUrl OneOff Created`**:
```json
{ "type": "SetVariable", "inputs": { "name": "varTargetSectionPagesUrl", "value": "@outputs('Create_Section_OneOff')?['body']?['pagesUrl']" } }
```
Pulls straight from the just-created section's own API response. Purely in-memory, no SharePoint call.

**`Create Page OneOff`**:
```json
{
  "type": "OpenApiConnection",
  "inputs": {
    "parameters": {
      "notebookKey": "Meeting Notes|$|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Meeting Notes",
      "sectionId": "@variables('varTargetSectionPagesUrl')",
      "pageContent": "<p class=\"editor-paragraph\">@{triggerBody()?['text_3']}</p>"
    },
    "host": { "connection": "shared_onenote-1", "operationId": "CreatePageInSection" }
  }
}
```
A plain OneNote page-creation call. No SharePoint write.

**This confirms, definitively, the full shape of the gap:** the one-off path creates OneNote sections and pages entirely in isolation from the SharePoint mapping list. It has no memory of anything it does. Every recapture of a one-off meeting effectively starts from zero.

## Working design, now reasonably well-evidenced

1. **Extend `Send an HTTP request to SharePoint`'s write** (or add an equivalent one-off write, mirroring it) to also populate `MeetingId` with `CalendarEventId` — for every meeting, recurring or one-off. This is a body-payload change to an existing action, not new infrastructure.
2. **Add a one-off equivalent of `Filter Existing Mapping`**, filtering the same list by `MeetingId = triggerBody CalendarEventId` instead of `SeriesMasterId`. Mirrors the existing recurring-path pattern closely.
3. **Wire that result into `varOutputPageSelfUrl Existing`'s currently-blank `value` field** — the Pattern 6 defect confirmed on 30 July — the same way the recurring path wires `Filter_Existing_Mapping`'s result through to `varFinalExistingPageSelfUrl`.
4. **Add a one-off write path** so that once a one-off meeting's page is first created, a mapping row gets written for it too (mirroring `Send an HTTP request to SharePoint`, keyed by `MeetingId` instead of `SeriesMasterId`) — otherwise step 2 will never find anything on a *second* capture of the same one-off meeting, since nothing would have written the row the first time.

Point 4 is the piece with no existing analogue to copy from directly (the recurring path's initial write happens during section resolution, which one-off meetings do have — `Condition Section Exists OneOff` — so the natural place to add the write is likely inside that condition's branches, mirroring where the recurring write sits under `Condition Section Exists Recurring`).

## `MeetingId`'s intended purpose — confirmed by David (31 July)

`MeetingId` was scoped for a planned future feature: finding post-meeting transcriptions, recordings, or action items and posting them into the correct OneNote meeting page. This is compatible with — arguably the same underlying need as — using it to resolve the existing page for one-off recapture: both require a stable identifier for "this specific meeting instance."

**One risk worth flagging, not yet resolved:** Teams transcript/recording retrieval (via Graph `onlineMeetings` / `callRecords`) sometimes keys off a **Teams online-meeting ID** (often derived from `joinWebUrl`), which is not always identical to the plain Outlook **calendar event ID** this doc has been calling `CalendarEventId`. If the future transcription feature specifically needs the Teams online-meeting ID rather than the calendar event ID, storing `CalendarEventId` in `MeetingId` now could mean that column holds the "wrong" identifier for that later feature, even though it's the right one for today's fix. Not a blocker — `CalendarEventId` is unambiguously correct for resolving the OneNote page — but worth a deliberate check (with whoever scoped the transcription feature, if not David alone) before treating `MeetingId` as permanently settled for both purposes.

## Still open / not yet confirmed

1. **The `Status`/empty-column discrepancy** noted above.
2. **Whatever action updates `PageSelfUrl` after page creation** (referenced but not yet directly inspected) — this is the natural template for point 4 above, since it's the existing pattern for "update a mapping row after a page now exists."
3. **Whether `CalendarEventId` (Outlook event ID) is the right long-term value for `MeetingId`**, given the Teams-online-meeting-ID risk noted above — may be worth revisiting once the transcription feature is actually scoped.

## Status

- **Evidence gathering essentially complete.** The shape of both the defect and a credible fix are now well-supported by direct Code view evidence, not inference from action names alone.
- **Still not a finalised, ready-to-build design.** The open items above should be resolved first — in particular, finding the `PageSelfUrl`-update action would directly template point 4's one-off write, removing the last piece of real uncertainty.
- **Recommended next step:** locate and inspect the `PageSelfUrl` post-creation update action (likely `HTTP Update SP PageSelfUrl`, seen in the Go to Operation list), then draft a concrete, numbered build plan referencing exact action names and exact expressions — at that point this becomes buildable rather than merely well-evidenced.
