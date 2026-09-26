# Flow Build Prompt — Teams → OneNote Meeting Capture

## Context
This is a rebuild of the existing Power Automate flows for the Teams → OneNote Meeting Capture system. The existing flows (Flow A, B, C, D) remain in production untouched. New flows are built alongside them and switched over only when fully tested end-to-end.

## Guiding principles — non-negotiable

1. **No SetVariable where a Compose will do.** SetVariable is only used when state must accumulate across loop iterations (e.g. appending to a string or array in a foreach loop). All other state is passed via Compose actions using `outputs('ActionName')` references downstream. This is the primary corruption prevention measure.

2. **One action, one responsibility.** No Compose or variable is shared between branches. If two branches need the same derived value, each branch gets its own Compose. Cross-branch references are the root cause of the most persistent bugs in the existing flows.

3. **IsRecurring gate explicit, not inferred.** Any branch that only applies to recurring meetings has `@equals(toLower(string(triggerBody()?['text'])), 'true')` as its first condition. Never rely on variable state to imply meeting type.

4. **Trigger and output contracts locked before building.** Every input field is named, typed, and marked required/optional before any action is added. Every response field is named and typed before the Response action is built. No mid-build contract changes.

5. **No dead code.** If a branch or action can never execute given the current trigger contract, it is not built. Document the decision in the known-good values reference instead.

6. **Scratch Diagnostics first.** Any expression involving nested functions, filter(), first(), or string manipulation longer than one line is proved in PA - Scratch Diagnostics before going into the production flow.

7. **GitHub updated after every Scope.** Trigger contract, output contract, known-good values, and Scope Peek Code are pushed to GitHub immediately after each Scope is confirmed green. Not at session end — after each Scope.

## Build format — every action must have
- **Reference code** (e.g. FB01, FB02a, SC01) — prefix matches the flow/Scope
- **Meaningful name** that describes what the action does, not what type it is
- **Expression or Dynamic Content only** — no manually typed connector field values
- All expressions verified in Peek Code before the next action is added

## Scope structure — mandatory
Every flow is wrapped in one outer Scope named `Scope_[FlowName]` containing all inner Scopes. Inner Scopes follow logical phases of work. This allows:
- Single Peek Code pull for full flow health check (outer Scope)
- Single Peek Code pull for one phase (inner Scope)
- Error handling at Scope level

Proposed inner Scope names are provided in each flow's build instructions. Additional Scopes may be added if a logical phase grows beyond ~8 actions.

## Corruption recovery — mandatory after every Scope
After every inner Scope is completed:
1. Peek Code the Scope → confirm all values present and match known-good reference
2. Save draft
3. Close and reopen the flow
4. Wait 20 seconds untouched
5. Run Flow Checker → must show 0 errors
6. Publish
7. Push Scope Peek Code to GitHub

Do not proceed to the next Scope until these 7 steps are complete.

## Known-good values reference
Maintained in GitHub at `flows/[flow-name]/known-good-values.md`. Written as each Scope is completed, not retrospectively.

| Ref | Action name | Value | Type | Last confirmed |
|---|---|---|---|---|

## Health check prompt (use at session start)
"Please perform a health check. Here is the Peek Code for Scope_[FlowName]: [paste]. Cross-reference all SetVariable and Compose values against the known-good values reference. Flag any blank or mismatched value. Do not suggest fixes or next steps until the health check is complete and I confirm I want to proceed."

## Session discipline
- One Scope per session where possible
- Never build on an unverified baseline
- If Flow Checker shows errors at session start, restore before building
- If corruption hits mid-session, stop, restore, re-verify before continuing
- Opus for design decisions and ambiguous structural choices
- Sonnet for build steps, expression writing, and known-good values documentation
