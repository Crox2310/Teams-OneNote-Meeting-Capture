# Flow B v2 Parent — Trigger Contract

**Locked:** 26 September 2026
**Constraint:** BYTE-IDENTICAL to Flow B v1. C10 must need zero rewiring at switchover.

## Trigger type

Unchanged from v1 (the Copilot-invoked flow trigger). **Do NOT convert the parent to PowerAppV2** — that would break C10's invoke binding. Only the internals of the flow change; the trigger keys, titles, required flags, and trigger type are frozen.

## Input fields (LOCKED — do not touch)

| Key | Title | Required | Role in v2 |
|---|---|---|---|
| `text` | IsRecurring | Yes | Carried for compatibility + echoed in output. **Not** used for routing. |
| `text_1` | MeetingTitle | Yes | Passed to child |
| `text_2` | SeriesMasterId | Yes | **The split signal** (`empty()` ⇒ one-off) |
| `text_3` | PageHtml | Yes | Passed to child |
| `text_4` | MeetingId | Yes | Passed to one-off child |
| `text_5` | OccurrenceDate | No | Passed to child |
| `text_6` | EndTime | No | Passed to child |
| `text_7` | JoinUrl | No | Passed to child |

## Routing

Split condition: `@empty(triggerBody()?['text_2'])`
- `true` ⇒ **one-off child**
- `false` ⇒ **recurring child**

`IsRecurring` (`text`) is deliberately not consulted (Decision 1). Its only v1 semantic role — the `RECURRING_SETUP_REQUIRED` guard — is absorbed structurally into the recurring child (which is recurring by construction).

## Parent → child input mapping

| Parent key | → Recurring child | → One-off child |
|---|---|---|
| `text` (IsRecurring) | — (dropped) | — (dropped) |
| `text_1` (MeetingTitle) | `text` | `text` |
| `text_2` (SeriesMasterId) | `text_1` | — (empty by definition) |
| `text_3` (PageHtml) | `text_3` | `text_3` |
| `text_4` (MeetingId) | — | `text_1` |
| `text_5` (OccurrenceDate) | `text_2` | `text_2` |
| `text_6` (EndTime) | `text_4` | `text_4` |
| `text_7` (JoinUrl) | `text_5` | `text_5` |
