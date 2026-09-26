# Build Methodology — Teams → OneNote Meeting Capture

**Created:** 26 September 2026
**Author:** David Croxson
**Context:** Distilled from the Flow A v2 rebuild session and the lessons learned across all prior build sessions on this project.

---

## What worked

The Flow A v2 build was completed in a single session with no corruption incidents, no rollbacks, and a working published flow by the end. This was a significant improvement over the v1 build history which had 12+ corruption incidents, multiple session-start recoveries, and several lost sessions. The following practices made the difference.

---

## The core principles

### 1. Lock contracts before building
Every input field and every output field is named, typed, and documented in `trigger-contract.md` and `output-contract.md` before a single action is added to the flow. No mid-build contract changes. This eliminated the cascading reference errors that affected v1 when `text_3`, `text_6`, `text_7` were added at different stages.

### 2. Compose-based state passing
All state is passed via `outputs('ActionName')` references between Compose actions. SetVariable and InitializeVariable are used only when state must accumulate across loop iterations. This is the primary corruption prevention measure — the platform's value-wipe corruption pattern specifically targets SetVariable and InitializeVariable actions. In v2 the entire flow has only two variables (both InitializeVariable, both top-level).

### 3. One action, one responsibility
No action is shared between branches. If two branches need the same derived value, each branch gets its own Compose. Cross-branch references were the root cause of the most persistent bugs in v1.

### 4. Scope structure
Every flow is wrapped in one outer Scope (`Scope_FlowA`) containing inner Scopes by logical phase. This gives:
- A single Peek Code pull for the full flow health check
- A single Peek Code pull for any one phase
- Self-documenting structure — each Scope has a description
- Error handling at Scope level

Inner Scope names follow the pattern: `Scope_CalendarFetch`, `Scope_CandidateResolution`, `Scope_NoMatch`, `Scope_SingleMatch`, `Scope_MultiMatch`, `Scope_Response`.

### 5. Reference codes on every action
Every action name starts with a reference code that is the first part of the name field, so copying and pasting the name field in the Designer gives the correctly prefixed action in one go. Format: `XX00 Descriptive Name` (e.g. `CA01 Compose DateContext`, `MM04b Compose Status Label`). Prefix matches the Scope (CA = CalendarFetch, CR = CandidateResolution, NM = NoMatch, SM = SingleMatch, MM = MultiMatch, RP = Response, TV = Top-level Variable).

### 6. Expressions only — never Dynamic Content picker
The Dynamic Content picker in the new Copilot Studio Designer inserts internal IDs (e.g. `outputs('builtinFunction-c95b8166-...')`) instead of action names. These break if an action is deleted and recreated. All expressions are typed manually in the Expression tab. This rule is non-negotiable.

### 7. Peek Code verification after every action
Every action is Peek Code verified before moving to the next one. The Peek Code is the authoritative view of what the platform has actually stored — the Parameters panel can show stale or loading values. If the Peek Code doesn't match the intended expression, fix it before proceeding.

### 8. Scratch Diagnostics first for complex expressions
Any expression involving nested functions, `filter()`, `first()`, array manipulation, or string operations longer than one line is proved in `PA - Scratch Diagnostics` before going into the production flow. This session confirmed that `indexOf()` does not support arrays in this environment — discovered in Scratch Diagnostics rather than mid-build.

### 9. GitHub updated after every Scope
The Scope Peek Code and known-good values are pushed to GitHub immediately after each Scope is confirmed green. Not at session end — after each Scope. This means if the session is interrupted, the next session has an authoritative reference for exactly where the build was.

### 10. No dead code
If a branch or action can never execute given the current trigger contract, it is not built. In v2 the FA15–FA26 IsSelectionMode branch from v1 (confirmed always `'NONE'`) was not rebuilt. The decision is documented in the trigger contract instead.

---

## The build format

The build instructions follow a Lego-set format:
- Each action is a numbered step with a reference code
- Every field is explicitly stated with its type (Expression / literal)
- Expressions are given in full, copy-pasteable
- Each Scope ends with mandatory corruption recovery steps (Peek Code → known-good values → save → close/reopen → Flow Checker → Publish → GitHub push)
- The build instructions are stored in GitHub before building starts, so they can be resumed after any interruption

---

## The health check protocol

At the start of every session:
1. Open the flow, wait 20 seconds untouched
2. Run Flow Checker — if errors appear, restore from known-good values before doing anything else
3. Peek Code the outer Scope (`Scope_FlowA`) and paste to the AI assistant with the prompt: *"Please perform a health check. Cross-reference all values against the known-good values reference. Flag any blank or mismatched value. Do not suggest fixes or next steps until the health check is complete and I confirm I want to proceed."*

For targeted phase review, peek the relevant inner Scope only.

---

## Platform constraints discovered during Flow A v2 build

These are confirmed behaviours of the Power Automate / Copilot Studio environment as of September 2026:

| Constraint | Detail | Workaround |
|---|---|---|
| `indexOf()` array support | `indexOf()` expects string as first parameter. Array not supported. | Use `varCandidateIndex` counter variable (InitializeVariable + IncrementVariable) |
| `InitializeVariable` nesting | Cannot be placed inside any Scope or Condition. Must be top-level in the flow. | Place all InitializeVariable actions before the outer Scope |
| Dynamic Content picker | Inserts internal action IDs not action names. References break on action recreation. | Always type expressions manually in the Expression tab |
| Trigger key assignment (new Designer) | New Copilot Studio Designer assigns input keys sequentially (`text`, `text_1`, `text_2`...) in the order inputs are added. | Add inputs in the correct order; document the resulting key mapping in trigger-contract.md |
| `filter()` in Compose | `filter()` is not available as a Compose expression. | Use Filter Array (Query) action instead |
| TV02 value wipe | `InitializeVariable` integer value wipes on Designer open (same pattern as v1 FA34A). | Session-start check: verify TV02 value = 1. Known-good value documented. |

---

## Model selection guidance

| Task | Model |
|---|---|
| Structural design decisions, cross-flow dependency analysis, ambiguous architecture choices | Opus |
| Trigger/output contract definition | Opus |
| Writing build instructions, expression writing, known-good values documentation | Sonnet |
| Step-by-step build execution, Peek Code verification, expression fixes | Sonnet |
| Corruption recovery (known values) | Sonnet |

Opus is used sparingly — only when genuine design ambiguity exists. The build itself is mechanical and Sonnet handles it well.

---

## What to do when corruption strikes mid-session

1. Stop immediately — do not attempt to fix forward
2. Run Flow Checker to identify which actions lost their values
3. Cross-reference against `known-good-values.md` for the exact restore values
4. Restore each action, Peek Code to confirm, before moving to the next
5. Flow Checker → Publish before continuing the build
6. Never stack a feature edit on top of an unrestored corruption

If more than 5 actions need restoring, restore from Version History instead of hand-restoring.

---

## File structure for a new flow build

```
flows/
  [flow-name]/
    trigger-contract.md       — locked before building
    output-contract.md        — locked before building
    build-instructions.md     — complete Lego build guide
    known-good-values.md      — populated as each Scope completes
    EXPRESSION-REFERENCE.md   — trigger key corrections and common expressions
    scope-peek-codes/
      README.md               — index of Scope files
      scope-[flow].md         — outer Scope (full flow)
      scope-[phase].md        — one file per inner Scope
```

Push the contracts and build instructions to GitHub **before starting the build**. Push each Scope's Peek Code immediately after it is confirmed green.

---

## The prompt that drives the build

See `flows/BUILD-PROMPT.md` for the master prompt used at the start of every new flow build session. This prompt encodes all the principles above and is pasted at the start of each build session to ensure consistency regardless of which AI session or model is used.
