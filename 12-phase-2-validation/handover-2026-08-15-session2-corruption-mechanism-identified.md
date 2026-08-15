# Handover — 15 August 2026 (session 2) — corruption mechanism identified

## ⏭ START HERE

This session continued directly from `handover-2026-08-15-URGENT-corruption-reproducible-session-paused.md`. The flow was restored to the 8 August 12:05 PM clean state at session start. **By the end of this session, the flow had corrupted again (26 errors) and has NOT yet been restored.** Check Version History and Flow Checker state before doing anything else.

---

## Biggest outcome this session: a concrete corruption mechanism hypothesis

David identified a pattern from memory of earlier build sessions that reframes this from "random platform corruption" to a specific triggering condition:

1. **Blank/empty values (`""`, whitespace, `[]`) do not commit cleanly or immediately** when entered into a connector/action field. There is a delay before it "settles".
2. **While that blank value is still unsettled, inserting a new connector/action elsewhere in the flow appears to disturb many other, unrelated, already-stable actions** — wiping their values too, not just the one being edited. This explains why one small change has repeatedly corrupted ~26 unrelated actions at once.
3. **Rebuilding sequentially from scratch, top to bottom, without ever leaving a field blank for more than one save** was previously found to avoid the issue entirely — but rebuilding from scratch is not practical for an existing flow that needs maintenance and future extension.

**Practical rules going forward, until Microsoft responds:**
- Never leave a field blank as an intermediate state. If a field needs to end up empty, don't hold that state across multiple saves — either don't create the action until the real value is ready, or use a non-blank placeholder and swap it in a separate, later save.
- Never insert a new connector in the same save as touching an existing blank field. Do them in strictly separate saves, with a Flow Checker pass and a pause in between.
- The four `InitializeVariable` actions in this flow (varTargetSectionPagesUrl x2, varOneNoteResolverResult x2) have no `value` field by design — they may be permanently sitting in the fragile "blank" state the mechanism describes, which could be why they appear disproportionately in every corruption incident's affected-action list.

---

## What happened this session, in order

1. **Confirmed clean state** at session start: Flow Checker 0 errors. Warnings read (0) rather than the expected baseline of 1 (the known harmless "Get items" OData warning) — confirmed on three separate checks this session. Unresolved why; worth watching next session.
2. **Captured Peek Code for all 26 known-at-risk actions** from the clean restore point, including the 4 `InitializeVariable` actions (no value field by design). Saved in full in `varOutStatus-backup-2026-08-15.md` (see below — still needs writing if not already present).
3. **Discovered `Set_varOutStatus` already read `value: "OK"`** — unexplained, no edit made this session to produce it. Confirmed via search that there is only one `SetVariable` action named `varOutStatus` in the flow — no hidden second branch. Bug 8 therefore may already be resolved, but this was NOT functionally confirmed via a successful test run reaching that action.
4. **Flow was found in Draft status, not Published**, when attempting a test run ("This flow is currently turned off"). The Workflows list could not toggle it on ("Classic workflows can't be enabled from this list") — had to publish from inside the flow instead. This turned out to be a UI/classic-workflow limitation, not a fifth corruption symptom.
5. **Published, then ran a test** with a deliberately novel MeetingTitle/MeetingId to force the new-section/one-off creation path. First attempt hit a transient BadGateway. Second attempt ran but failed at `Create_Page_OneOff` with **empty `sectionId`** in the raw inputs — this is Bug 5, reproduced live with full run diagnostics. The run never reached `Set_varOutStatus`, so Bug 8 still has no functional confirmation.
6. **Immediately after this test run, with no field edit made in between, Flow Checker went from 0 to 26 errors** — the exact same 26 actions as previous incidents. Screenshotted in full across 5 scrolled views. **This is Incident 4 in the corruption log.** The timing — right after a publish event plus a recently-set, unconfirmed value change (`varOutStatus` = `OK`) — fits the blank/structural-edit race condition hypothesis above, though not a perfect match (no new connector was inserted this time — the trigger may be broader than "insert a connector" specifically, possibly any publish/structural event near an unsettled value).
7. **Drafted the Microsoft support ticket** (`MICROSOFT-SUPPORT-TICKET-DRAFT-2026-08-15.md`) covering all four dated incidents, the blank/structural-edit mechanism hypothesis, and the publish/Flow Checker independence finding. **Not yet submitted to Microsoft.**

---

## Status of core fixes

- **Bug 7** (recurring second-capture) — unchanged, presumed still fixed, not re-tested this session.
- **Hyperlink truncation fix** — unchanged, presumed still fixed, not re-tested this session.
- **Bug 8** (`varOutStatus` empty) — field value now reads `OK` but **unconfirmed** functionally; origin of the change is unknown. Treat as unresolved until a test run actually reaches and confirms this action.
- **Bug 5** (one-off recapture, empty `sectionId`) — still unfixed, now reproduced live with full run diagnostics this session. Root cause confirmed in a later session: `Create_Page_OneOff`'s `sectionId` parameter is `@variables('varTargetSectionPagesUrl')`, and that variable is set exclusively by the corrupted SetVariable actions. Bug 5 and the corruption pattern are the same root cause, not two separate bugs.
- **Microsoft support ticket** — drafted in full this session, still not submitted. Now four dated incidents deep. Submitting this should be the first action of next session, before any further editing attempts.

---

## Immediate next steps for next session

1. **Check current Flow Checker state first.** Session ended mid-corruption (incident 4, 26 errors), unresolved.
2. **Submit the Microsoft ticket** (`MICROSOFT-SUPPORT-TICKET-DRAFT-2026-08-15.md`) before attempting any further fixes — we have strong, fresh evidence right now.
3. **Restore via Version History** to the last confirmed-clean point — do not manually re-enter 26 values by hand. Use the Peek Code backup file only as a verification reference after restoring, not as the primary recovery method.
4. **Apply the new blank-value / structural-edit separation rule** to any further Bug 8 fix — once restored, if `varOutStatus` reads blank again, fix it in its own isolated save, with no other structural change nearby, and wait/confirm settled before doing anything else.
5. **Once stable, run a test that actually reaches `Set_varOutStatus`** (avoid triggering Bug 5's path if possible, or fix Bug 5 first) to get a genuine functional confirmation of Bug 8.
6. Investigate Bug 5's `sectionId` wiring gap once stable.

---

**Status:** Flow currently corrupted (26 errors), published state uncertain, session paused mid-incident. Microsoft ticket drafted but not submitted. Restore first, submit ticket second, then resume Bug 8/Bug 5 work using the new isolation rule.
