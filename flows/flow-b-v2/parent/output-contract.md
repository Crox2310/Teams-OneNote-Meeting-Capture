# Flow B v2 Parent — Output Contract

**Locked:** 26 September 2026
**Constraint:** BYTE-IDENTICAL keys to Flow B v1 `Respond to the Agent`. Verified against live v1 Response node (20 fields).

All fields `type: string`, `statusCode: 200`, key order preserved as v1.

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

Only one child ever runs. For each child-owned field:
```
@{coalesce(body('RL_Call_Recurring')?['<key>'], body('RL_Call_OneOff')?['<key>'])}
```
The skipped branch's action returns null; coalesce selects the populated one.

## Graceful child-failure (Decision 3)

`outstatus` carries a terminal `'ERROR'` fallback so a failed/absent child body still yields a clean status:
```
@{coalesce(body('RL_Call_Recurring')?['outstatus'], body('RL_Call_OneOff')?['outstatus'], 'ERROR')}
```
The relay Scope runs after the router Scope with runAfter = [`Succeeded`, `Failed`] so a child failure inside the Condition does not abort the parent before the Respond. Parent-echo fields still populate from the trigger on the error path.
