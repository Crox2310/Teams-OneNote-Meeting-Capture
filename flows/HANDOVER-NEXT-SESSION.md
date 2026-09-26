# Handover Prompt — Next Session

**Project:** Teams → OneNote Meeting Capture
**GitHub:** Crox2310/Teams-OneNote-Meeting-Capture
**Date of last session:** 26 September 2026

---

## What happened this session

Flow B v2 split architecture was fully designed and build instructions written for two of the three flows:

- **Architecture, contracts, and Scope maps** locked for all three flows (parent + recurring child + one-off child) in `flows/flow-b-v2/`.
- **Build instructions complete** for the **parent** (`flows/flow-b-v2/parent/build-instructions.md`) and the **one-off child** (`flows/flow-b-v2/oneoff/build-instructions.md`).
- **Build instructions partial** for the **recurring child** (`flows/flow-b-v2/recurring/build-instructions.md`) — Steps 1–4 complete (trigger, outer scope, Normalize, MappingLookup); Steps 5–12 blocked on two missing v1 inputs (see below).

---

## Current state

**Flow A v2:** Published, live, working. `candidatesjson` fix still outstanding (number selection broken for multi-match).
**Flow B v1:** Still live, untouched. Known corruption issues.
**Flow B v2:** Architecture locked, build instructions written, **nothing built yet in Power Automate**.
**Flow C:** Published, live.
**Flow D:** Published, live.
**Topic:** Calling Flow A v2. C10 still calling Flow B v1 unchanged.

---

## Build order for the morning

### 1. One-off child first (fully specified, proves child-flow plumbing cheaply)

Follow `flows/flow-b-v2/oneoff/build-instructions.md` step by step.

Before you can build it, satisfy the child-flow plumbing prerequisites:
- Trigger must be **PowerApps (V2)**
- Terminal action: **"Respond to a PowerApp or Flow"**
- After publishing: set connectors to **"Use this connection"** in Run-only-users on `make.powerautomate.com`
- Flow must be in a solution to appear in the Run-a-Child-Flow picker

### 2. Parent (fully specified, small — ~1/4 session)

Follow `flows/flow-b-v2/parent/build-instructions.md`.

Prerequisites:
- One-off child must be published and solution-aware before the parent's Run-a-Child-Flow actions will work
- Recurring child does not need to exist yet — the parent's RT03a Run Recurring Child action can be wired once the recurring child is built

### 3. Recurring child (Steps 1–4 only until v1 inputs are available)

Follow `flows/flow-b-v2/recurring/build-instructions.md` up to the **⏸ STOP HERE** marker.

To unblock Steps 5–12, paste the following to the AI at the start of that session:
1. Peek Code of `Apply_to_each_Existing_Section` from **live Flow B v1** (already have the guard — also need `Filter_Existing_Section_By_Name` and the section-create/use branch nesting before it in the new-mapping path)
2. Expression from live v1 `Compose_AgentResponseSummary`

---

## Two things to confirm at build time (flagged in the files)

1. **Recovery-create write-back** — v1 has a latent gap: when the existing-mapping guard creates a page because the stored one is gone, it does not write the new URL back to the SharePoint row. v2's WriteBack scope runs on any `PageAction=Created` and fixes this. Confirm you want the fix, or restrict WriteBack to new-mapping rows only for strict v1 parity. Flagged in `recurring/scope-map.md`.

2. **`Compose_AgentResponseSummary` expression** — not in the known-good reference. Pull from live v1 before building Step 10 of the recurring child. If it is user-facing, reproduce it faithfully; if diagnostic only, a placeholder is fine.

---

## Topic change needed at switchover

When Flow B v2 parent is published and ready:
- In the Meeting Capture Topic, rewire **C10** to call `PA - Meeting Capture - B - Router - v2` instead of v1. C10's input keys (`text` through `text_7`) and the parent's output field bindings are byte-identical to v1 — **no other Topic changes needed**.
- Confirm the Topic's user-facing message for `outstatus = 'STALE_MAPPING'` vs `'SUCCESS'` — v2 moves successful existing-section captures from STALE→SUCCESS (v1 was accidentally mislabelling them). If the Topic shows a distinct message for STALE, those users will now see the SUCCESS message instead.

---

## Key file locations added this session

```
flows/flow-b-v2/
  README.md                        architecture, decisions 1-7, findings from recheck
  parent/
    trigger-contract.md            LOCKED — byte-identical to v1
    output-contract.md             LOCKED — 20 fields, byte-identical keys
    scope-map.md                   Scope_FlowB_Parent → Scope_Router + Scope_Relay
    build-instructions.md          COMPLETE — all steps
  oneoff/
    trigger-contract.md            LOCKED
    output-contract.md             LOCKED
    scope-map.md                   Scope_FlowB_OneOff → 7 inner scopes
    build-instructions.md          COMPLETE — all steps
  recurring/
    trigger-contract.md            LOCKED
    output-contract.md             LOCKED
    scope-map.md                   Scope_FlowB_Recurring → 7 inner scopes + OutStatus precedence
    build-instructions.md          PARTIAL — Steps 1-4 complete, Steps 5-12 blocked on v1 inputs
  scope-peek-codes/               (to be populated as each Scope confirms green)
```

---

## Session-start protocol (every flow, every session)

1. Open the flow, wait 20 seconds untouched.
2. Run Flow Checker — if errors appear, restore from known-good before touching anything else.
3. Peek Code the outer Scope (`Scope_FlowB_Parent` / `Scope_FlowB_Recurring` / `Scope_FlowB_OneOff`) and cross-reference against GitHub known-good values.

---

## Model guidance

- **Build execution** (following these instructions, Peek Code verification, expression fixes, corruption recovery): **Sonnet**
- **Completing recurring child Steps 5–12** once v1 Peek Code is available: **Sonnet** (structure is defined in scope-map.md; it is transcription work at that point)
- **Any new design ambiguity** that emerges during the build: **Opus**
