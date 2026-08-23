# Finding — Topic checker false positives on C2_Call_FlowA_Initial

**Status:** Investigated and closed. Confirmed as a checker-level display issue, not a functional defect. No fix applied — none needed.

**Discovered:** 23 August 2026, evening, during a routine Topic checker pass unrelated to any active development (development had been paused for the day).

---

## Symptom

Topic checker on `Meeting Capture (v4 rebuild)` reports 3 persistent errors, all attached to the same action, `C2_Call_FlowA_Initial` (the first of two calls to Flow A in the Topic):

1. **Action: "Flow not found or is turned off"**
2. **Condition: "Variable is being set to an incorrect type. Assigned: String, expected: Unspecified"**
3. **Condition: "Variable is being set to an incorrect type. Assigned: String, expected: Unspecified"** (a second instance)

## Investigation, in order, with evidence at each step

1. **Full Topic YAML reviewed.** `invokeFlowAction_eBUGn8` (the `C2_Call_FlowA_Initial` node) is structurally sound — correct `flowId` (`d9d7ccf7-7d61-f111-a826-6045bde03856`, Flow A), correct input/output bindings, `text_1: NONE` (a documented literal sentinel matching Flow A's own `FA15_Compose_IsSelectionMode` logic). No YAML defect found.
2. **Connection Settings checked.** All three connections (Flow A, Flow B, and the third utility flow) showed green "Connected" status at time of check.
3. **A duplicate, disabled "Meeting Capt..." topic (Activity-triggered) was found in the Topics list and deleted** as genuine housekeeping — but the 3 errors persisted identically afterward, ruling this out as the cause.
4. **Clicked into each error in Topic checker**, confirming all 3 anchor to the same single action, `C2_Call_FlowA_Initial` — not scattered across multiple nodes as initially assumed. The action's card in the canvas displays correctly: the connected flow ("PA - Resolve Meeting Selection...") resolves and links via "View flow details," and all 9 output bindings display correctly typed as `string` in the UI.
5. **A fresh Publish was performed** (7:00 PM, 23 Aug) specifically to test whether this was a stale schema-cache issue that a full republish would clear. **Errors persisted, identically, post-publish.**
6. **Live functional test performed**: a real recurring-meeting capture was run through the published agent. **It completed successfully end-to-end.**

## Conclusion

The Topic checker errors on `C2_Call_FlowA_Initial` are **false positives** — a display/validation artifact in Copilot Studio's Topic checker, not a real defect in the Topic, the flow, or the connection. The action works correctly at runtime, confirmed by a successful live capture performed *after* the errors were observed and *after* a fresh publish.

**Likely underlying cause** (unconfirmed, not investigated further given no functional impact): Topic checker's static schema validation may be comparing against a stale or differently-inferred snapshot of Flow A's Response action output types, out of sync with the actually-published, currently-working flow. This is consistent with — though not confirmed to be the same root cause as — the connection instability David has separately observed ("green now, red/stale every now and then" in Connection Settings).

## Why no fix was applied

- The YAML, connections, and live functional test all confirm nothing is actually broken.
- The only actions available to try clearing the checker display (re-selecting the flow in the picker, deleper edits to the binding) would touch `C2_Call_FlowA_Initial` directly — a working, correctly-functioning action — for zero functional benefit and real corruption risk, per the established pattern documented throughout this project's incident log.
- Per the working method established all day: verify with evidence before acting, and don't fix what isn't actually broken.

## Recommendation for future sessions

- **Do not attempt to "fix" this by editing `C2_Call_FlowA_Initial`** unless a genuine functional failure is observed and traced back to this specific action via Activity trace — at that point, evidence would justify investigation. Until then, treat these 3 Topic checker errors as known, harmless, and expected.
- **Worth adding to the Microsoft support brief** (`microsoft-discussion-brief-corruption-bug.md`) as a second, distinct symptom category alongside the SetVariable/Compose corruption pattern: Topic checker reporting persistent false-positive errors on a functioning `InvokeFlowAction` node, surviving a full republish. Different symptom, possibly related root cause (environment-level schema/connection sync instability) — worth mentioning together given they may share a root cause even though they manifest differently.
- If this ever needs re-investigating, start from this doc rather than re-running the full investigation from scratch — the YAML, connection, and duplicate-topic angles have already been ruled out.

---
*Investigated and closed 23 August 2026.*
