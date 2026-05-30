# Shared Journey Contract vFinal

## Flow A status values

| Status | Meaning | Topic route |
|---|---|---|
| `SINGLE_MATCH` | One meeting resolved | Route to one-off or recurring path |
| `MULTIPLE_MATCHES` | More than one candidate found | Route to UJ2 |
| `NO_MATCH` | No meeting found | Route to UJ5 |
| `ERROR` | Flow A could not safely resolve | Safe error; do not call Flow B |

## Flow B OutStatus values

| OutStatus | Meaning | Primary use |
|---|---|---|
| `SUCCESS` | OneNote write and required persistence completed | UJ1/UJ3/UJ4 |
| `RECURRING_SETUP_REQUIRED` | Recurring mapping is missing, stale, or unsafe | UJ3 → UJ4 |
| `PARTIAL_SUCCESS` | OneNote page was created but mapping persistence failed | UJ4 |
| `SETUP_SECTION_NOT_FOUND` | Existing section lookup returned zero matches | UJ4 |
| `SETUP_SECTION_AMBIGUOUS` | Existing section lookup returned multiple matches | UJ4 |
| `ERROR` | Flow B could not complete safely | Safe error |

## Required topic routing and retry variables

| Topic variable | Type | Purpose |
|---|---|---|
| `Topic.FlowAStatus` | Text | Primary routing signal from Flow A. Sourced from `FlowA.Status`. Used in every routing condition and Flow B call gate. |
| `Topic.SectionRetryCount` | Number | UJ4 section-name retry tracking. Initialise to `0`. Maximum value `1` before graceful exit. |

## Core topic variable type rules

| Topic variable | Type | Rule |
|---|---|---|
| `Topic.SelectedNumber` | Number | Convert to text before passing to Flow A: `Text(Topic.SelectedNumber)` |
| `Topic.SectionChoiceNumber` | Number | Compare as number; derive text `SectionChoice` |
| `Topic.OneOffFallbackChoice` | Number | Compare as number |
| `Topic.RecoveryChoiceNumber` | Number | Compare as number |
| `Topic.MatchCount` | Text | Convert to number for bounds check: `Int(Topic.MatchCount)` |
| `Topic.IsRecurring` | Text | Compare as string: `Topic.IsRecurring = "true"` |
| Inclusion flags | Text | Compare as strings, for example `Topic.IncludeLocation = "true"` |
| Flow A/Flow B outputs | Text | Treat all as strings unless explicitly captured by topic as Number |

## V1 Outlook data inclusion flags

Use text values `true` and `false`, not booleans.

| Topic variable | Type | Default | Purpose |
|---|---|---|---|
| `Topic.IncludeMeetingBody` | Text | `false` | Include full event body in OneNote |
| `Topic.IncludeAttendeesDetail` | Text | `false` | Include full attendee detail |
| `Topic.IncludeLocation` | Text | `true` | Include location if available |
| `Topic.IncludeOrganizer` | Text | `true` | Include organiser summary if available |
| `Topic.IncludeOnlineMeetingDetails` | Text | `false` | Include Teams/join detail if available |
| `Topic.IncludeAttachmentSummary` | Text | `false` | Include attachment summary if available |
| `Topic.IncludeAttachmentContent` | Text | `false` | Reserved for V2; must remain false in V1 |

## Universal Flow B call gate

```text
Topic.FlowAStatus = "SINGLE_MATCH"
AND Topic.MeetingTitle is not empty
AND Topic.CalendarEventId is not empty
AND Topic.PageHtml is not empty
```
