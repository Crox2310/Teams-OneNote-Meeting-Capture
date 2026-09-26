# Flow B v2 Recurring Child — Trigger Contract

**Locked:** 26 September 2026
**Trigger type:** PowerAppV2 ("When a Power App or flow calls this flow") — required for Run-a-Child-Flow. Terminal action is **"Respond to a PowerApp or Flow"** (not the Skills response; that is the parent's).

The child's keys are its own; they need no relation to the parent's keys. `IsRecurring` and `MeetingId` are dropped — this child is recurring by construction.

## Input fields (LOCKED)

| Key | Title | Required | From parent | v1 equivalent key |
|---|---|---|---|---|
| `text` | MeetingTitle | Yes | parent `text_1` | v1 `text_1` |
| `text_1` | SeriesMasterId | Yes | parent `text_2` | v1 `text_2` |
| `text_2` | OccurrenceDate | Yes | parent `text_5` | v1 `text_5` |
| `text_3` | PageHtml | Yes | parent `text_3` | v1 `text_3` |
| `text_4` | EndTime | No | parent `text_6` | v1 `text_6` |
| `text_5` | JoinUrl | No | parent `text_7` | v1 `text_7` |

## ⚠ Key-remap warning (critical)

The child keys are renumbered relative to v1. The same literal `text_N` means different things:

| v1 key | v1 meaning | Same key in this child |
|---|---|---|
| `text_1` | MeetingTitle | SeriesMasterId |
| `text_2` | SeriesMasterId | OccurrenceDate |
| `text_5` | OccurrenceDate | **JoinUrl** |

**Do not copy v1 expressions verbatim.** The Normalize scope re-exposes every trigger input as a named Compose (`NZ00a..`), and every downstream expression references that Compose, never a raw `triggerBody()?['text_N']`. Port v1 logic by swapping to the Compose names, not by carrying the key numbers.

## Plumbing prerequisites
- Terminal "Respond to a PowerApp or Flow" action.
- All connectors set to "Use this connection" in Run-only-users.
- Flow must be solution-aware to appear in the Run-a-Child-Flow picker.
