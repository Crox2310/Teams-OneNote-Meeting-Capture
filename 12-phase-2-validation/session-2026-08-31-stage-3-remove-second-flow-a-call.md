# Session close — Stage 3

**Date:** 31 August 2026
**Flows changed:** Flow A — FA12B Select action, candidatesjson output field.
**Topic changed:** Yes — selection rebuilt, cancel fix, EndDialog, C6F added, second flow call removed.
**Flow B changed:** No.

## What was built

### S3.0 — ParseJSON feasibility test

Throwaway Topic `ZZ - Scratch ParseJSON Test` confirmed: `ParseJSON` exists in Copilot Studio Power Fx, `Index()` is 1-based, untyped-object field access resolves through `Text()`. Output: `Title: Meeting Two | Id: BBB222`.

### S3.1 — FA12B Select Candidates

New Select action after `FA11`, before `FA13`. Replaces the dead-code `FA12_Append_to_array_varCandidates` (expression was stored as a literal string, never evaluated, nothing consumed it).

Final map after three correction iterations:

- `Title`: `@{coalesce(item()?['subject'], '')}`
- `Start`: `@{coalesce(item()?['start'], '')}` — flat string, not nested
- `Id`: `@{coalesce(item()?['id'], '')}`
- `IsRecurring`: `@{if(empty(coalesce(item()?['seriesMasterId'], '')), 'false', 'true')}`
- `SeriesMasterId`: `@{coalesce(item()?['seriesMasterId'], '')}`
- `OnlineMeetingUrl`: `""` — connector returns no `onlineMeeting` object; pre-existing gap
- `BodyPreview`: `@{if(empty(coalesce(item()?['body'], '')), '', substring(coalesce(item()?['body'], ''), 0, min(2000, length(coalesce(item()?['body'], '')))))}` — guarded for empty bodies

Connector shape findings: `start` is a flat string; no `onlineMeeting` object; no `type` field; bodies can be empty strings. Array size at runtime: ~11KB for 7 candidates.

### S3.2 — candidatesjson output field

New field in `FA43_Respond_to_agent`: `"candidatesjson": "@{string(body('FA12B_Select_Candidates'))}"`

### S3.3 and S3.4 — Topic rebuild

Four changes published atomically:

1. `C2_Call_FlowA_Initial` — new output binding `candidatesjson: Topic.CandidatesJson`
2. `C6D_Check_Number` — condition gains range check; body replaced with `SetMultipleVariables` resolving all six fields via `Index(ParseJSON(Topic.CandidatesJson), Value(Topic.TopicSelectedNumber)).FieldName`
3. `C6F_Check_Number_OutOfRange` — new branch after C6D; catches out-of-range numbers; message: "That number isn't in the list. Please pick a number between 1 and {Topic.MatchCount}."
4. `EndDialog` added to `C4C_Check_Cancel` and `C6E_Check_Cancel` — fixes pre-existing fall-through bug causing a second Flow A call and BadGateway after cancel. This error was misidentified as a platform intermittent issue five times during today's testing.
5. `invokeFlowAction_bIIKPf` (second Flow A call) removed.

Deliberate behaviour change: meeting created between list display and selection will not be reflected. Accepted — seconds-wide window, saves a full calendar round-trip.

## Gate results

| Test | Scenario | Result |
|---|---|---|
| 1 | Multi-match, pick 3 (SC&L FLT Stand-up, recurring) | PASS |
| 2 | Pick 99 out of range | PASS — correct message, reprompt |
| 3 | Type C at selection prompt | PASS — cancel message only, EndDialog working |
| 4 | Recurring meeting (covered by test 1) | PASS |
| 5 | P navigation then pick from new list | PASS — CandidatesJson rebinds on GotoAction |
| 6 | Zero-match day via 20/10/28 then N | PASS — C4B navigation intact |

## Build errors (three, all on FA12B)

1. Nested shape assumption — `start.dateTime` doesn't exist; connector returns flat `start`
2. Unguarded substring on empty body — four of seven test-day events had empty bodies, visible in raw output that had just been read
3. False ID mangling alarm — comparison was against `webLink` (URL-encoded) not `id`; IDs in candidatesjson are correct

## Dead code and gaps found

- `FA11`/`FA12` — dead code, never evaluated, remove in Stage 6
- `FA12` IsRecurring used `type` field which doesn't exist in this connector shape
- `OnlineMeetingUrl` — always `''`, pre-existing; extracting from body HTML is a separate backlog item
- `FA29B_Compose_OutBodyPreview_Single` — same unguarded substring pattern; lower risk path; add to Stage 6

## Next action

Stage 5 — Perceived latency. Check `Create_OneNote_Page` connector action against S0.1 findings first.

*Session closed 31 August 2026.*
