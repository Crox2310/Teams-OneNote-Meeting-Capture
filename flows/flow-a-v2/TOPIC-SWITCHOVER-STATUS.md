# Topic Switchover — Status and Next Steps

**Last updated:** 26 September 2026

## Current state

Flow A v2 is built, published and tested. The Topic switchover to v2 is partially complete but blocked.

## What was done

- v2 connected as a Tool in the agent ✔
- Connection Settings refreshed (both flows Connected) ✔
- Topic YAML updated with correct flowId (`12b1d0c8-a6b9-f111-aaaf-7ced8d745465`) ✔
- C2 action re-added through UI pointing at v2 ✔
- `candidatesjson` removed from C2 output bindings (not in v2 contract) ✔

## Current blocker

Copilot Studio is not switching the flow binding via YAML flowId change alone. The canvas consistently shows `PA - Resolve Meeting Selection` (v1 name) rather than v2, and Topic checker shows 14 PowerFxErrors on the SetMultipleVariables block.

Root cause: when C2 was deleted and re-added through the UI, the new action was assigned a new internal ID (`invokeFlowAction_rLKsKC`). All GotoAction references in the Topic still point at the old ID (`invokeFlowAction_eBUGn8`). Restoring the old ID via YAML doesn't force the flow binding to update — Copilot Studio stores the connection binding separately.

## Fix required in next session

Two options:

**Option A (recommended) — Accept new ID, update all GotoAction references:**
1. Let C2 keep its new ID (`invokeFlowAction_rLKsKC`)
2. In the YAML, do a find-and-replace: all `actionId: invokeFlowAction_eBUGn8` → `actionId: invokeFlowAction_rLKsKC`
3. There are 7 GotoAction references to update (goto_C4C_P, goto_C4C_N, goto_C4C_Date, CHYopX, I8IQMw, Tb9vLh, and the implicit one from C1)
4. Save → Topic checker → should clear all errors

**Option B — Keep old ID, force binding via UI:**
1. In the C2 action on canvas, use the flow picker to explicitly select v2
2. This should update the connection binding without changing the action ID
3. Save → Topic checker

Option A is cleaner because it accepts the new ID rather than fighting the platform.

## Known gap logged

C6D (number selection path) uses `ParseJSON(Topic.CandidatesJson)` which relies on `candidatesjson` output from C2. Since v2 doesn't output `candidatesjson`, the number selection path will not work after switchover. This needs a separate fix — either add `candidatesjson` to v2's output contract (requires a new Compose in Scope_Response), or redesign the selection resolution to use a different approach.

This is a known gap and does not block the initial switchover — the no-match and multi-match candidate list display will work; only the step where a user selects a number from the list will fail.

## v2 trigger contract reminder

When C2 is correctly bound to v2, the input keys are:
- `text` = InSelectedNumber (value: `NONE`)
- `text_1` = DateContext (value: `=Topic.DateContext`)

These differ from v1 which used `text_1` and `text_3`.
