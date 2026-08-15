# Handover — 15 August 2026 (session 2, part 2) — 26-action recovery complete, published, root cause of publish-only failure found

## ⏭ START HERE

Continuing from `handover-2026-08-15-session2-corruption-mechanism-identified.md`, which left the flow corrupted (Incident 4, 26 errors) mid-session. **This part of the session recovered the flow, published it successfully, and found a new root-cause pattern.** As of this handover: **Flow Checker shows 0 errors, 0 warnings, flow is published.** Confirm this is still true at the start of next session before doing anything else — this flow's stability has not held for more than a few hours at a time recently.

---

## What happened, in order

1. **Two more corruption incidents occurred during recovery** — logged as Incident 5 and Incident 6 below.
2. **Despite that, all 26 previously-corrupted actions were successfully written back** to their correct values, one isolated action at a time (per-save Flow Checker verification), using the reference values compiled in `flow-reference-2026-08-15-full-peek-code-capture.md`.
3. **A `Save draft` succeeded with 0 errors / 1 warning** (the expected healthy baseline) — but **`Publish` then failed** with a distinct, structural error not seen before this session:

   > `InvalidTemplate`: "The inputs of template action 'varTargetSectionPagesUrl' ... is invalid. Action 'Apply_to_each' must be a parent 'foreach' scope of action 'varTargetSectionPagesUrl' to be referenced by 'repeatItems' or 'items' functions."

4. **This revealed a new corruption sub-pattern.** Draft-save validation does not catch cross-scope reference errors; publish validation does. This means a flow can appear clean via Flow Checker + draft save while still containing a structural fault that only surfaces at publish time — consistent with, but distinct from, the previously-documented "publish succeeds while Flow Checker shows errors underneath" finding (see 15 August ticket draft). This is effectively the inverse case: Flow Checker clean, draft save clean, **publish itself is the only check that catches it.**

5. **Root cause traced and fixed.** The flow's top-level `InitializeVariable` action for `varTargetSectionPagesUrl` (positioned early in the flow, well before `Condition IsRecurring` even evaluates, and structurally outside any `Apply_to_each`/`For_each_1` loop) had somehow acquired a `value` field: `@items('Apply_to_each')?['pagesUrl']`. Every other `InitializeVariable` action in this flow has no `value` field at all — this one picking up a stray, out-of-scope reference is almost certainly a corruption artifact, likely a value cross-wired from the *correct* place this expression belongs (inside `Apply_to_each`, on the properly-scoped `varTargetSectionPagesUrl 1` action, which does legitimately use this exact expression and was confirmed correct).
6. **Fix applied**: cleared the Value field on the top-level `InitializeVariable` action entirely (via Parameters tab — Code view is read-only in this environment), restoring it to `name` + `type` only, matching every other InitializeVariable action.
7. **Flow Checker re-checked: 0 errors, 1 warning. Publish attempted again: succeeded.** ("Your agent flow was published and optimized for rapid execution.")
8. **Final Flow Checker check post-publish: 0 errors, 0 warnings** (the "Get items" OData warning no longer appearing at all at this final check — another minor state fluctuation worth noting, not investigated further this session).

---

## Corruption incident log — additions this session

### Incident 5 — 15 August 2026, ~16:21
Occurred with **zero edits made** between a confirmed clean check (0 errors, 1 warning, right after successful Batch 1 write-back of 4 actions) and this event. Flow Checker jumped to **21 errors** across the recurring-branch and page-creation-branch actions (Batch 3/4 — none of which had been touched yet this session). The 4 just-fixed one-off-branch actions were NOT in this error list, suggesting that specific fix held. Screenshotted in full.

### Incident 6 — 15 August 2026, shortly after Incident 5
After a single, correct, isolated edit to `OF05b — Set varFinalPageDecision (OneOff)`, error count **increased** from 15 to 19 — the wrong direction for an isolated correct fix. A **new error type** appeared for the first time this session: `"Invalid parameters"` (distinct from the usual `'Value' is required'`) on three untouched recurring-branch actions and on `OF05c` (not yet edited). This is a different corruption signature than incidents 1–5 and should be flagged distinctly to Microsoft — it suggests corruption isn't limited to blanking values; it can also produce a different validation-error class on actions with intact-but-now-invalid values.

Despite Incident 6's escalation, continued single-action edits **did not** trigger further escalation — the very next check after continuing showed the error count drop sharply (19 → 1), and shortly after, full recovery to 0/1. This is inconsistent with a strict "editing while corrupted makes it worse" rule and suggests the corruption events themselves may be somewhat decoupled from the specific edit that immediately precedes them — worth flagging to Microsoft as an open question rather than a settled mechanism.

---

## New root-cause finding: publish-only validation gap

**This is the most actionable technical finding of the session and should be added to the Microsoft ticket.**

- Flow Checker (via the panel) and `Save draft` do **not** validate cross-scope expression references (e.g., an action outside a `foreach` referencing `items('LoopName')`).
- `Publish` **does** run this validation, and fails with a clear, specific `InvalidTemplate` error identifying the exact action and the exact scope violation.
- This means a flow can pass every visible check (Flow Checker 0 errors, successful draft save) while carrying a structural fault that **only surfaces at publish time** — a second, distinct instance of the "checks don't agree with each other" problem already flagged in the ticket draft (the first being "publish succeeds while Flow Checker shows errors").
- Practical implication for future sessions: **do not treat a clean Flow Checker + successful draft save as sufficient confirmation that a flow is genuinely fixed.** Attempt a publish (or at minimum, be aware publish may still fail) before considering recovery complete.

---

## Status of core fixes (end of session)

- **All 26 originally-corrupted actions**: confirmed fixed, values verified against `flow-reference-2026-08-15-full-peek-code-capture.md`, flow published successfully.
- **Additional fix this session**: top-level `varTargetSectionPagesUrl` InitializeVariable action's stray out-of-scope value cleared — this was not one of the original 26, found only because it blocked Publish specifically.
- **Bug 8** (`varOutStatus`): both the InitializeVariable and the final SetVariable read `OK`. Still not functionally confirmed via a test run reaching the final `Set_varOutStatus` action — this remains the top open item for next session.
- **Bug 5** (one-off recapture, empty `sectionId`): root cause confirmed in a prior part of this session — `Create_Page_OneOff`'s `sectionId` comes from `variables('varTargetSectionPagesUrl')`, which is set by actions that were corrupted. With today's fixes now in place, Bug 5 may be resolved as a side effect — **not yet re-tested.**
- **Microsoft support ticket**: still not submitted. Now six dated incidents (four from before this session, plus Incidents 5 and 6 today), plus the new publish-only validation gap finding and the new "Invalid parameters" error-type variant from Incident 6. This is now overdue by two weeks and should be the first action of next session.

---

## Recommended next steps for next session

1. **Confirm Flow Checker still shows 0/0 or 0/1** before touching anything.
2. **Submit the Microsoft ticket** — update `MICROSOFT-SUPPORT-TICKET-DRAFT-2026-08-15.md` with Incidents 5 and 6, the new error-type variant, and the publish-only validation gap finding before submitting.
3. **Run a test that reaches `Set_varOutStatus`** to get genuine functional confirmation of Bug 8 (use test data likely to route through the existing-page-update path, or the fixed new-section-creation path, to avoid unrelated failures).
4. **Re-test Bug 5's scenario** (one-off recapture, new section) now that the underlying variable-setting actions are fixed.
5. Continue treating every single edit as a potential corruption trigger — screenshot Flow Checker before and after any change, however small.

---

**Status: flow is currently clean and published (0 errors, 0 warnings at last check). Do not make speculative edits before confirming this state holds. Submit the Microsoft ticket next, before further feature or bug work.**
