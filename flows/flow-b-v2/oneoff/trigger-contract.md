# Flow B v2 One-Off Child — Trigger Contract

**Locked:** 26 September 2026
**Trigger type:** PowerAppV2 ("When a Power App or flow calls this flow") — required for Run-a-Child-Flow. Terminal action is **"Respond to a PowerApp or Flow"** (not the Skills response; that is the parent's).

`SeriesMasterId` is dropped (empty by definition for one-off). `IsRecurring` dropped.

## Input fields (LOCKED)

| Key | Title | Required | From parent | v1 equivalent key |
|---|---|---|---|---|
| `text` | MeetingTitle | Yes | parent `text_1` | v1 `text_1` |
| `text_1` | MeetingId | Yes | parent `text_4` | v1 `text_4` |
| `text_2` | OccurrenceDate | Yes | parent `text_5` | v1 `text_5` |
| `text_3` | PageHtml | Yes | parent `text_3` | v1 `text_3` |
| `text_4` | EndTime | No | parent `text_6` | v1 `text_6` |
| `text_5` | JoinUrl | No | parent `text_7` | v1 `text_7` |

## ⚠ Key-remap warning (critical)

Child keys are renumbered relative to v1 (e.g. v1 `text_5` = OccurrenceDate, but here `text_5` = JoinUrl; v1 `text_4` = MeetingId, but here `text_1` = MeetingId). **Do not copy v1 expressions verbatim.** The Normalize scope re-exposes every trigger input as a named Compose (`NZ00a..`); every downstream expression references the Compose, never a raw `triggerBody()?['text_N']`.

## Plumbing prerequisites
- Terminal "Respond to a PowerApp or Flow" action.
- All connectors set to "Use this connection" in Run-only-users.
- Flow must be solution-aware to appear in the Run-a-Child-Flow picker.
