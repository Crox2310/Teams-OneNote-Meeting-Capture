# Flow B v2 Parent — Scope Map

**Locked:** 26 September 2026
**Variables:** none. **SetVariable:** none.

Prefix scheme: `RT` = Router, `RL` = Relay.

## Outer Scope: `Scope_FlowB_Parent`

Wraps everything so a single Peek Code pull is the full-flow health check.

### Inner Scope: `Scope_Router` (RT)
Responsibility: derive the route once, branch, call exactly one child.
- `RT01 Compose Route Key` — `@if(empty(triggerBody()?['text_2']), 'ONEOFF', 'RECURRING')` (single source of truth for the split).
- `RT02 Condition Route` — **branch selector** (below).
- `RT03a Run Recurring Child` (false branch) — Run-a-Child-Flow; passes MeetingTitle/SeriesMasterId/OccurrenceDate/PageHtml/EndTime/JoinUrl per the parent→child mapping.
- `RT03b Run OneOff Child` (true branch) — Run-a-Child-Flow; passes MeetingTitle/MeetingId/OccurrenceDate/PageHtml/EndTime/JoinUrl.

### Inner Scope: `Scope_Relay` (RL)
Responsibility: assemble the 20-field v1 output shape and respond. runAfter `Scope_Router` = [`Succeeded`, `Failed`] (Decision 3).
- `RL01..` Compose relays — four parent echoes direct from trigger; sixteen child-relay fields via coalesce across `RT03a_Run_Recurring_Child` / `RT03b_Run_OneOff_Child` with `''` fallback (see output-contract.md).
- `RL_Respond` — **Skills "Respond to the agent"** (matching v1's `kind: Skills`), byte-identical 20 keys, `statusCode: 200`. NOT "Respond to a PowerApp or Flow."

## Branch selector — `RT02 Condition Route`
```
@equals(outputs('RT01_Compose_Route_Key'), 'ONEOFF')
```
- **True** ⇒ `RT03b Run OneOff Child`
- **False** ⇒ `RT03a Run Recurring Child`

## Peek Code strategy
One outer `Scope_FlowB_Parent` pull covers the entire flow. Session start: open, wait 20s, Flow Checker, Peek Code outer Scope, cross-reference known-good.
