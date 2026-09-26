# Flow B v2 Parent — Output Contract

**Locked:** 26 September 2026
**Constraint:** BYTE-IDENTICAL keys to Flow B v1 `Respond to the agent` (Skills Response). Verified against live v1 Response node (20 fields).

All fields `type: string`, `statusCode: 200`, key order preserved as v1. The parent's terminal action is the Skills **"Respond to the agent"** — not "Respond to a PowerApp or Flow."

## Field ownership

Four fields are pure trigger echoes and are owned by the **parent** (it already holds these values at its own trigger). The remaining sixteen are computed and **relayed from whichever child ran**.

| # | Key | Owner | v2 value source |
|---|---|---|---|
| 1 | `outisrecurring` | Parent echo | `@{triggerBody()?['text']}` |
| 2 | `outmeetingtitle` | Parent echo | `@{triggerBody()?['text_1']}` |
| 3 | `outseriesmasterid` | Parent echo | `@{triggerBody()?['text_2']}` |
| 4 | `outpagehtml` | Parent echo | `@{triggerBody()?['text_3']}` |
| 5 | `outspitemcount` | Child relay | child `outspitemcount` |
| 6 | `outmatchcount` | Child relay | child `outmatchcount` |
| 7 | `outbranchresult` | Child relay | child `outbranchresult` (Decision 7: mirrors match count) |
| 8 | `outonenoteresolverresult` | Child relay | child `outonenoteresolverresult` |
| 9 | `outtargetsectionpagesurl` | Child relay | child `outtargetsectionpagesurl` |
| 10 | `outcreatedpagelink` | Child relay | child `outcreatedpagelink` |
| 11 | `outcreatedpageselfurl` | Child relay | child `outcreatedpageselfurl` |
| 12 | `outfinaltargetsectionpagesurl` | Child relay | child `outfinaltargetsectionpagesurl` (= field 9) |
| 13 | `outresolverresult` | Child relay | child `outresolverresult` (= field 8) |
| 14 | `outexistingpageselfurl` | Child relay | child `outexistingpageselfurl` |
| 15 | `outpagedecision` | Child relay | child `outpagedecision` |
| 16 | `outpageroute` | Child relay | child `outpageroute` (string `True`/`False`) |
| 17 | `outpageaction` | Child relay | child `outpageaction` |
| 18 | `outupdatehtmlfragment` | Child relay | child `outupdatehtmlfragment` |
| 19 | `outagentresponsesummary` | Child relay | child `outagentresponsesummary` |
| 20 | `outstatus` | Child relay | child `outstatus` |

**Known v1 redundancies preserved:** field 12 duplicates 9; field 13 duplicates 8; field 7 mirrors 6. Kept as-is for zero behaviour change; de-dup is a future cleanup, not this rebuild.

## Relay binding pattern

The two child-call actions are `RT03a_Run_Recurring_Child` and `RT03b_Run_OneOff_Child` (in `Scope_Router`). Only one runs; the skipped one's body is null. For each of the 16 child-owned fields, with a `''` terminal fallback so the error path never emits literal null:
```
@{coalesce(body('RT03a_Run_Recurring_Child')?['<key>'], body('RT03b_Run_OneOff_Child')?['<key>'], '')}
```
This coalesce-across-branches is the same pattern v1 already uses (e.g. `Set_varOutputPageLink_Existing`).

## Graceful child-failure (Decision 3)

`outstatus` uses a terminal `'ERROR'` (not `''`) so a failed/absent child still yields a routable status:
```
@{coalesce(body('RT03a_Run_Recurring_Child')?['outstatus'], body('RT03b_Run_OneOff_Child')?['outstatus'], 'ERROR')}
```
`Scope_Relay` runs after `Scope_Router` with runAfter = [`Succeeded`, `Failed`] so a child failure inside the Condition does not abort the parent before the Response. Parent-echo fields still populate from the trigger on the error path.

## Byte-fidelity notes (reproduce v1 expression *form*, not just intent)

- **`outpageroute`** — v1 = `@{equals(variables('varFinalPageDecision'), 'PAGE_EXISTS')}` (boolean string-interpolated). The child must interpolate the boolean the same way, `@{equals(outputs('<PageDecision>'), 'PAGE_EXISTS')}`, so the casing v1 emits is preserved — do **not** substitute a reworded `string()` that might differ in case.
- **`outspitemcount`** — v1 = `@{int(coalesce(outputs('Compose_SP_Item_Count'), 0))}` (int interpolated to string). Reproduce the `int(coalesce(..., 0))` form.
