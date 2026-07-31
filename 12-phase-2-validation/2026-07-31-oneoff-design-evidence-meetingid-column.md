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

## Working hypothesis for the design decision

**The mapping list plausibly already anticipates a second identifier beyond `SeriesMasterId` — and `CalendarEventId` is a strong candidate for what belongs in the existing, unused `MeetingId` column.**

If this hypothesis holds, the design becomes significantly simpler than "candidate 2" as originally scoped on 30 July (which assumed a wholly new mechanism would need to be built). Instead:

1. **On mapping write** (whatever action currently writes new rows to this list — likely `Send an HTTP request to SharePoint`, not yet inspected in detail): also write `CalendarEventId` into the existing `MeetingId` column, for every meeting (recurring or one-off), alongside the existing `SeriesMasterId` write.
2. **On one-off recapture:** add a `Filter_Existing_Mapping`-equivalent step for the one-off path, filtering the same list by `MeetingId = trigger's CalendarEventId` instead of `SeriesMasterId`.
3. **Wire the result into `varOutputPageSelfUrl Existing`'s currently-blank `value` field** (the Pattern 6 defect from 30 July), the same way the recurring path wires `Filter_Existing_Mapping`'s result into `varFinalExistingPageSelfUrl`.

This would reuse the existing list, existing columns, and an existing identifier already flowing through the system — rather than requiring new storage or a new identifier scheme.

## Explicitly not yet confirmed — needed before this becomes a real design, not just a hypothesis

1. **What currently writes to the mapping list at all**, and whether it already populates any of the "empty" columns for some other row we haven't seen (the one row viewed is the only one currently in the list, so this can't be cross-checked yet against a second example).
2. **Whether `MeetingId` was already intended for exactly this purpose** by whoever originally designed the list, versus being vestigial/for a different unimplemented feature. Worth asking David directly if he recalls the original intent — much faster than reverse-engineering it.
3. **The one-off section-resolution actions** (`Get Sections OneOff`, `Filter OneNote Section OneOff`, etc.) still need their Code view reviewed to confirm they don't already partially handle this in some way not yet observed.
4. **The `Status`/empty-column discrepancy** noted above.

## Status

- **Not a fix. Not a finalised design.** This is evidence that narrows the design space meaningfully: reusing the existing `MeetingId` column with `CalendarEventId` as the one-off key looks like a strong, low-new-build option, but needs the open questions above answered before committing to it.
- **Next step:** confirm what writes to the mapping list (to see whether `MeetingId` population is trivial to add), and ask David directly whether `MeetingId`'s original purpose is known/documented anywhere, before drafting the actual build plan.
