# Flow A Test Pack

## Test FA-T01 — No valid meetings

### Inputs

| Input | Value | Value type |
|---|---|---|
| `InUserSearchText` | `capture notes for my meeting` | Plain text |
| `InDateContext` | `today` | Plain text |
| `InMaxCandidates` | `5` | Plain text |

### Expected output

```json
{
  "Status": "NO_MATCH",
  "MatchCount": "0",
  "CandidateList": "",
  "MeetingTitle": "",
  "CalendarEventId": "",
  "IsRecurring": "false",
  "SeriesMasterId": "",
  "Start": "",
  "End": "",
  "Location": "",
  "AttendeesSummary": "",
  "OutErrorMessage": ""
}
```

## Test FA-T02 — One valid meeting

### Inputs

| Input | Value | Value type |
|---|---|---|
| `InUserSearchText` | `capture notes for my meeting` | Plain text |
| `InDateContext` | `today` | Plain text |
| `InMaxCandidates` | `5` | Plain text |

### Expected output

```json
{
  "Status": "SINGLE_MATCH",
  "MatchCount": "1",
  "CandidateList": "",
  "MeetingTitle": "Populated from Outlook subject",
  "CalendarEventId": "Populated from Outlook id",
  "IsRecurring": "false or true based on event type",
  "SeriesMasterId": "Populated if available, otherwise empty string",
  "Start": "Populated if start.dateTime available",
  "End": "Populated if end.dateTime available",
  "Location": "Populated if location.displayName available",
  "AttendeesSummary": "",
  "OutErrorMessage": ""
}
```

## Test FA-T03 — Multiple valid meetings

### Inputs

| Input | Value | Value type |
|---|---|---|
| `InUserSearchText` | `capture notes for my meeting` | Plain text |
| `InDateContext` | `today` | Plain text |
| `InMaxCandidates` | `5` | Plain text |

### Expected output

```json
{
  "Status": "MULTIPLE_MATCHES",
  "MatchCount": "2 or more as string",
  "CandidateList": "1. Meeting A — date, start–end\n2. Meeting B — date, start–end\n",
  "MeetingTitle": "",
  "CalendarEventId": "",
  "IsRecurring": "false",
  "SeriesMasterId": "",
  "Start": "",
  "End": "",
  "Location": "",
  "AttendeesSummary": "",
  "OutErrorMessage": ""
}
```

## Test FA-T04 — Recurring meeting present

### Inputs

| Input | Value | Value type |
|---|---|---|
| `InUserSearchText` | `capture notes for my weekly meeting` | Plain text |
| `InDateContext` | `today` | Plain text |
| `InMaxCandidates` | `5` | Plain text |

### Expected output

```json
{
  "Status": "SINGLE_MATCH or MULTIPLE_MATCHES depending on calendar day",
  "IsRecurring": "true if type is occurrence or exception",
  "SeriesMasterId": "Populated if seriesMasterId or iCalUId is available; otherwise empty string"
}
```

## Debug validation

Inspect `FA03A_DEBUG_RawConnectorOutput` and confirm actual connector fields before changing expressions.
