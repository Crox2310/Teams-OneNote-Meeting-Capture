# Copilot Studio Topic Branching

## Topic responsibility

The Topic owns the user journey. The Topic calls Agent Flows and routes the user based on returned string outputs.

## Flow A call inputs

| Flow A input | Value | Value type |
|---|---|---|
| `InUserSearchText` | User request text | Dynamic content |
| `InDateContext` | `today` | Plain text |
| `InMaxCandidates` | `5` | Plain text |

## Flow A output mappings

| Flow A output | Topic variable |
|---|---|
| `Status` | `Topic.FlowAStatus` |
| `MatchCount` | `Topic.MatchCount` |
| `CandidateList` | `Topic.CandidateList` |
| `MeetingTitle` | `Topic.MeetingTitle` |
| `CalendarEventId` | `Topic.CalendarEventId` |
| `IsRecurring` | `Topic.IsRecurring` |
| `SeriesMasterId` | `Topic.SeriesMasterId` |
| `Start` | `Topic.Start` |
| `End` | `Topic.End` |
| `Location` | `Topic.Location` |
| `AttendeesSummary` | `Topic.AttendeesSummary` |
| `OutErrorMessage` | `Topic.OutErrorMessage` |

## Branching

```text
If Topic.FlowAStatus = "NO_MATCH"
    → no-match recovery message

If Topic.FlowAStatus = "SINGLE_MATCH"
    → proceed to one-off or recurring path based on Topic.IsRecurring

If Topic.FlowAStatus = "MULTIPLE_MATCHES"
    → show CandidateList and ask user to select a number

If Topic.FlowAStatus = "ERROR"
    → safe error message
```
