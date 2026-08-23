# Naming Convention Audit & Standardisation — Rollout Prep

**Status:** Not started. Scoping doc only — no renames have been made. Created 23 August 2026, ahead of a wider rollout, to ensure the platform is maintainable by people without the full 10-week build history in their head.

**Why this matters for rollout specifically:** every naming reference in this repo (`known-good-values-master-reference.md`, `known-good-values-flow-a-reference.md`, the amendment log, every session note) was written by someone who built the flow action-by-action and can hold the current inconsistency in their head. A new person picking this up cold — for support, for extension, or for learning — doesn't have that context. Clear, consistent naming is part of the platform being genuinely robust, not just functionally correct.

---

## Current state, assessed 23 August 2026

| Layer | Convention | Assessment |
|---|---|---|
| **Flow A** | `FA##_Verb_Noun`, numbered in execution order (`FA08_Get_calendar_view_of_events`, `FA09B_Filter_ExcludeLeaveAndPeriodEntries`, `FA13_Compose_MatchCount`) | **Strong.** Execution order and purpose both readable from the name alone. Held up well under fast iteration on 23 Aug (FR-01/FR-02) — minimal Peek Code needed just to orient. |
| **Topic** | Mostly `C##_Verb_Noun` (`C1_Set_DateContext`, `C4_Check_MatchCount`, `C6C_Check_Date`), but several nodes retain auto-generated canvas IDs never renamed (`invokeFlowAction_bWHHeg`, `sendActivity_VCuFOo`, `conditionGroup_k0eXvc`) | **Decent, uneven.** The numbered nodes are good; the un-renamed auto-generated ones are opaque and inconsistent with the rest. |
| **Flow B** | No consistent scheme. At least three co-existing styles have grown organically over ~10 weeks: plain descriptive (`Compose_SafeSectionName`, `Filter_Existing_Mapping`), a prefixed one-off-branch convention (`OF05a_—_Set_varFinalExistingPageSelfUrl_(OneOff)`), and bare auto-incremented duplicates with no descriptive suffix (`varTargetSectionPagesUrl_1`, `varTargetSectionPagesUrl_2`) | **Weakest of the three.** No numbering, so execution order must be inferred from `runAfter` chains rather than read off the name. The `_1`/`_2` bare-suffix pattern is the specific worst case — two actions doing branch-specific work with names that don't say which branch. |

---

## Why this is scoped separately from a normal bug/feature session

Renaming actions in a live Power Automate flow is **structural surface area**, not a content change:

- Any rename risks breaking a `runAfter` reference or an `outputs('OldName')`/`body('OldName')` expression elsewhere in the same flow, or — critically — **in the other flow or the Topic**, since Flow A's `FA43_Respond_to_agent` output field names are consumed by the Topic, and Flow B's `Respond`/`Response` action likewise.
- This is precisely the kind of edit that has triggered platform corruption before (see `known-good-values-master-reference.md`'s incident log — large-scale renames/restructures correlate with corruption hits more than small value edits do).
- Every reference document in this repo (`known-good-values-master-reference.md`, `known-good-values-flow-a-reference.md`, `amendment-log.md`, every dated session note) uses today's action names. A rename pass makes all of that **historically accurate but no longer directly copy-pasteable** against the live flow, unless the docs are updated in the same pass.

Given that, this should be its own dedicated session, not squeezed into other work, with the docs updated as part of the same pass rather than as a follow-up.

---

## Proposed scope, when this is picked up

1. **Agree the target convention first**, before touching anything. Recommendation (not yet agreed): extend Flow A's `FA##_Verb_Noun` pattern to Flow B as `FB##_Verb_Noun`, numbered in execution order, with branch context folded into the verb/noun rather than a bare numeric suffix (e.g. `FB33_Set_OneOffSectionPagesUrl_Existing` instead of `varTargetSectionPagesUrl_1`). Topic nodes get the remaining un-renamed auto-generated IDs brought in line with the existing `C##_Verb_Noun` pattern.
2. **Full fresh Peek Code capture of Flow A, Flow B, and Topic YAML** before any renaming starts — this becomes the pre-rename baseline, same evidence-first discipline as every other change this repo has made.
3. **Map every cross-reference** before renaming anything: every `runAfter`, every `outputs('X')`/`body('X')` call, every Topic `InvokeFlowAction` `output.binding` field name that depends on Flow A/B's Response action field names. A rename that only touches the action's own display name is low-risk; a rename that changes a Response schema field name is high-risk because the Topic depends on it by exact field name.
4. **Rename in the lowest-risk order**: internal-only actions first (no cross-flow dependency), then flow-internal `runAfter` chains, then — only if genuinely needed — Response/output field names that the Topic depends on, since those are the highest-blast-radius changes.
5. **One rename (or small batch), then Flow Checker, then Peek Code re-verification, then next batch** — same discipline as every build this repo has done, not a bulk find-and-replace.
6. **Update every reference doc in the same session** — `known-good-values-master-reference.md`, `known-good-values-flow-a-reference.md`, and add a note to `amendment-log.md` recording the rename pass as its own amendment (or set of amendments), so future readers aren't left cross-referencing old names against a renamed flow.
7. **Live end-to-end test after renaming** — new capture, recapture, recurring, one-off, P/N/date navigation — before considering the rename pass complete, exactly as every other change this repo has made was verified.

---

## Not yet decided

- The exact target naming convention for Flow B (proposal above is a starting point, not agreed).
- Whether Topic node `id` fields (the YAML-level identifiers, distinct from `displayName`) get renamed too, or only left as auto-generated IDs with clean `displayName`s — renaming `id` fields is likely lower-value and higher-risk than renaming `displayName`s, since `id` is sometimes referenced by `GotoAction.actionId`.
- Whether this happens before or after the Microsoft support ticket is finally submitted, since a renamed flow changes what any future Microsoft-side investigation would see compared to what's documented in the ticket's incident log — worth sequencing deliberately rather than by accident.

---
*Created 23 August 2026. Scoping only. Pick up as its own dedicated session ahead of rollout — do not fold into routine bug/feature work given the structural risk involved.*
