# Microsoft support ticket draft — Power Automate flow corruption: SetVariable/InitializeVariable value fields blanking on save

Status: DRAFT — ready for submission, not yet submitted.
Overdue since: 1 August 2026. This is the fourth dated incident without a ticket having been filed.

---

## Summary

A cloud flow in Power Automate (built via Copilot Studio Designer) repeatedly and reproducibly loses the `value` field on multiple `SetVariable` and `InitializeVariable` actions simultaneously — typically 21–26 actions at once — producing a wave of "Value is required" Flow Checker errors across actions that were not touched in the triggering edit. This has now happened on four separate dated occasions across two weeks, with a precise, repeatable trigger identified on the most recent occasion.

A second, arguably more serious finding: **publishing a flow does not run Flow Checker validation.** A version was published with a green "success" banner while secretly carrying 25 corrupted actions underneath — meaning this corruption can reach production undetected, not just the editing draft.

---

## Environment

- **Product**: Copilot Studio / Power Automate cloud flow (classic workflow type)
- **Environment**: Agents — Personal Productivity (environment ID `76f9c3bd-16c5-e540-8bb4-7171f4745b45`)
- **Flow name**: PA — Resolve OneNote Meeting Section — v2 Clean Build
- **Flow ID**: `ed112c88-b94b-f111-bec6-002248a38052`
- **Connectors involved**: SharePoint, OneNote (Business), Teams trigger ("When an agent calls the flow")
- **Affected action types**: `SetVariable`, `InitializeVariable`
- **Browser / client**: Chrome, macOS

---

## The core bug: blank/empty value fields + structural edits = mass corruption

Across multiple build sessions on this flow we have observed a consistent mechanism:

1. **Entering a blank/empty value** into an action's field — whether as `""`, whitespace, or an empty array `[]` — does not commit cleanly or immediately. There is a noticeable delay before the field / connector "settles" into a stable state.
2. **While a blank value is still unsettled**, if a **structural edit** is made elsewhere in the same flow — most notably *inserting a new action/connector* — the save event appears to disturb many other, unrelated, already-stable actions' value fields, wiping them to empty.
3. The blast radius is disproportionate: a single new action insertion has corrupted up to 26 unrelated values in one save.
4. **Building sequentially from scratch, top to bottom, never leaving a field blank for more than one save**, has been the only reliable workaround found so far — but is impractical for maintaining or extending an existing, already-built flow.

This points toward a **race condition in the save/validation pipeline**, where a field recently set to blank has not yet fully committed server-side when a concurrent structural save (adding/inserting an action) occurs, causing the serializer to write out blank values for other actions that were never actually edited.

---

## Incident log (dated, reproducible)

### Incident 1 — 1 August 2026
Corruption observed following edits during active build session. See `handover-2026-08-01-corruption-incident-and-fix-list.md` in repo.

### Incident 2 — 2 August 2026 (overnight/session 4)
Corruption recurred. See `handover-2026-08-02-session4-overnight-corruption-and-fixes.md`.

### Incident 3 — 8 August 2026
Corruption struck while attempting a single-field edit on `Set_varOutStatus`. Restored via Version History to 8 August 12:05 PM. See `handover-2026-08-08-SESSION-CLOSEOUT.md` and related 8 August handovers.

### Incident 4 — 15 August 2026 (this session, fully documented)
**This is the strongest repro case to date.**

- Session started from a confirmed-clean restore (8 August 12:05 PM) — Flow Checker: 0 errors, 0 warnings confirmed at 10:12, again at "no errors found" screens multiple times across the session.
- **Baseline mismatch noted early:** the expected baseline was 0 errors, 1 harmless "Get items" OData warning — but Warnings read (0) on every check this session. Possibly unrelated, but flagged as another state discrepancy.
- A `Set_varOutStatus` action was found to already contain `value: "OK"` unexplained — not blank, as the handover expected, and no edit had been made this session to produce it.
- Attempted a test run to functionally confirm this value; flow was found in **Draft status, not Published**, and could not be enabled from the Workflows list ("Classic workflows can't be enabled from this list" — requires publishing from inside the flow instead).
- After publishing and running a test, the run failed on a separate, already-known issue (Bug 5 - empty `sectionId` on one-off recapture).
- **Immediately afterward, with no field edit made by the user between checks**, Flow Checker went from 0 errors to **26 errors**, all `'Value' is required`, across the exact same set of SetVariable/InitializeVariable actions previously identified as at-risk (see full list below). Screenshots captured at the time.

**This incident fits the mechanism above exactly**: a test run had just been executed against a published version containing an unexplained/unconfirmed value change (`Set_varOutStatus` = `"OK"`), and the flow had just transitioned Draft → Published shortly before — consistent with a recent structural/publish event coinciding with a recently-set value not yet fully settled.

---

## Affected actions (26, this incident)

`varTargetSectionPagesUrl` (×2, InitializeVariable), `varOneNoteResolverResult` (×2, InitializeVariable), `Set varPageAction Created`, `Set varOutputPageSelfUrl Created`, `Set varOutputPageLink Created`, `Set varPageAction Created OneOff`, `Set varOutputPageSelfUrl Created OneOff`, `Set varOutputPageLink Created OneOff Gate`, `Set varPageAction ExistsNoCreate`, `Set varOutputPageSelfUrl Existing`, `Set varPageAction UpdatedAppend`, `Set varOutputPageLink Existing`, `Set varOutputPageLink Created OneOff`, `varFinalExistingPageSelfUrl` (InitializeVariable), `varFinalPageDecision` (InitializeVariable), `varFinalMatchCount` (InitializeVariable), `Set varTargetSectionPagesUrl OneOff Exists`, `Set varOneNoteResolverResult Exists OneOff`, `Set varTargetSectionPagesUrl OneOff Created`, `Set varOneNoteResolverResult Created OneOff`, `OF05a — Set varFinalExistingPageSelfUrl (OneOff)`, `OF05b — Set varFinalPageDecision (OneOff)`, `OF05c — Set varFinalMatchCount (OneOff)`, `Set varOutStatus`.

Notably, **the InitializeVariable actions in this list never had a `value` field in their original definition** (they only declare `name` and `type`) — so Flow Checker demanding a value for these specifically is itself part of the anomaly, not a normal validation state.

---

## Request to Microsoft

1. Investigate whether a race condition exists in the Designer save pipeline between (a) a recently-set blank/empty value field not yet fully committed server-side, and (b) a concurrent structural save (inserting/editing another action).
2. Confirm whether Flow Checker validation is guaranteed to run at publish time, given we observed a successful publish on a version that Flow Checker separately flagged as 25 errors.
3. Advise whether `InitializeVariable` actions without an explicit `value` field are expected to be flagged by Flow Checker as requiring a value, or whether this is itself a symptom of the corruption.
4. Advise on a safe workflow for applying single-field edits to an existing flow without triggering this pattern — in particular whether a full solution export/import (editing `definition.json` outside the live Designer surface) avoids the trigger.

---

## Supporting evidence available on request

- Screenshots of the 26-error Flow Checker state (15 August, ~10:21–10:22).
- Full Peek Code capture of all 26 affected actions from the confirmed-clean restore point, prior to this incident (see `varOutStatus-backup-2026-08-15.md` in this folder).
- Prior handover docs for incidents 1–3 (listed above).
- Run history entry showing BadRequest on `Create_Page_OneOff` with empty `sectionId` (separate issue, Bug 5, possibly related root cause).

---

*Drafted 15 August 2026. Update this file with the actual Microsoft ticket number once submitted, and link it from the next session's handover.*
