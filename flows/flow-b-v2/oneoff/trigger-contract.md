# Flow B v2 One-Off Child — Trigger Contract

**Locked:** 26 September 2026
**Trigger type:** PowerAppV2 ("When a Power App or flow calls this flow") — required for Run-a-Child-Flow.

`SeriesMasterId` is dropped (empty by definition for one-off). `IsRecurring` dropped.

## Input fields (LOCKED)

| Key | Title | Required | From parent |
|---|---|---|---|
| `text` | MeetingTitle | Yes | parent `text_1` |
| `text_1` | MeetingId | Yes | parent `text_4` |
| `text_2` | OccurrenceDate | Yes | parent `text_5` |
| `text_3` | PageHtml | Yes | parent `text_3` |
| `text_4` | EndTime | No | parent `text_6` |
| `text_5` | JoinUrl | No | parent `text_7` |

## Plumbing prerequisites
- Terminal Respond action.
- All connectors set to "Use this connection" in Run-only-users.
- Flow must be solution-aware to appear in the Run-a-Child-Flow picker.
