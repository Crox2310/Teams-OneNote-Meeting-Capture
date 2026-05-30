# User Journey 5 — No Match / Recovery

## Purpose

UJ5 handles no-match recovery. It prevents Flow B from being called when Flow A has not resolved a meeting.

## Recovery menu

```text
1. Search today's meetings again with different wording
2. Stop
```

## Clean handoff note

UJ5 does not implement UJ4 recurring setup.

UJ5 only routes to UJ4 if a retry later resolves a valid single recurring meeting without a reliable `SeriesMasterId`.

## Flow B block rule

Do not call Flow B on:

```text
NO_MATCH
ERROR
MULTIPLE_MATCHES
blank MeetingTitle
blank CalendarEventId
```
