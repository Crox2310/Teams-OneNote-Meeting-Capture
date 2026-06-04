# UJ1 Validation Record — One-off Single Match

## User journey

UJ1 — one-off single match

## Purpose

Validate that the user-facing Meeting Capture solution can resolve a single one-off Outlook meeting and create a OneNote meeting notes page without creating a recurring SharePoint mapping.

## Expected behaviour

- Flow A returns SINGLE_MATCH.
- MatchCount equals 1.
- MeetingTitle is populated.
- CalendarEventId is populated.
- IsRecurring is false.
- Topic routes to SINGLE_MATCH.
- Flow B follows the one-off path.
- OneNote page is created in the default meeting notes section.
- No recurring SharePoint mapping is created.
- Agent returns a clear success response.

## Test input

Record the exact user prompt used here:

```text
[Add test prompt]
