# Microsoft support ticket draft — Power Automate flow corruption: SetVariable/InitializeVariable value fields blanking on save, plus a distinct publish-only validation gap

Status: DRAFT — ready for submission, not yet submitted.
Overdue since: 1 August 2026. This is the sixth dated incident without a ticket having been filed.

---

## Summary

A cloud flow in Power Automate (built via Copilot Studio Designer) repeatedly and reproducibly loses the `value` field on multiple `SetVariable` and `InitializeVariable` actions simultaneously — typically 15–26 actions at once — producing a wave of "Value is required" Flow Checker errors across actions that were not touched in the triggering edit. This has now happened on **six** separate dated occasions across two weeks, with a precise, repeatable trigger identified on some occasions and a zero-edit variant on others.

A second corruption signature was observed for the first time on 15 August: instead of blanking values, an error type of **"Invalid parameters"** appeared on actions whose values were still intact but apparently no longer considered valid — distinct from the usual "Value is required" on blanked fields.

A third, separate and arguably more serious finding: **publishing a flow does not always agree with Flow Checker or draft-save validation, in either direction.** We have observed:
- Publish succeeding with a green banner while Flow Checker separately showed 25 errors underneath (production can silently receive corrupted state).
- The reverse: Flow Checker showing 0 errors and a draft save succeeding cleanly, while Publish itself failed with a structural `InvalidTemplate` error that neither Flow Checker nor draft-save had caught. Root cause was a cross-scope expression reference (an action outside a `foreach` loop referencing `items('LoopName')`) that had been silently written into an action's `value` field — most likely a corruption artifact, since that specific `InitializeVariable` action should never have had a `value` field at all (see Incident 4 addendum below).

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
3. The blast radius is disproportionate: a single edit has corrupted up to 26 unrelated values in one save.
4. **Building sequentially from scratch, top to bottom, never leaving a field blank for more than one save**, has been the only reliable workaround found so far — but is impractical for maintaining or extending an existing, already-built flow.
5. **This mechanism is not fully sufficient to explain all observed incidents** (see Incidents 5 and 6 below, which occurred without a qualifying structural edit) — we believe it captures a real and significant trigger, but not the only one.

This points toward a **race condition in the save/validation pipeline**, where a field recently set to blank has not yet fully committed server-side when a concurrent save occurs, causing the serializer to write out blank (or, in at least one case, cross-wired/incorrect) values for other actions that were never actually edited by the user.

---

## Incident log (dated, reproducible)

### Incident 1 — 1 August 2026
Corruption observed following edits during active build session. See `handover-2026-08-01-corruption-incident-and-fix-list.md` in repo.

### Incident 2 — 2 August 2026 (overnight/session 4)
Corruption recurred. See `handover-2026-08-02-session4-overnight-corruption-and-fixes.md`.

### Incident 3 — 8 August 2026
Corruption struck while attempting a single-field edit on `Set_varOutStatus`. Restored via Version History to 8 August 12:05 PM. See `handover-2026-08-08-SESSION-CLOSEOUT.md` and related 8 August handovers.

### Incident 4 — 15 August 2026, morning session
- Session started from a confirmed-clean restore (8 August 12:05 PM) — Flow Checker: 0 errors confirmed multiple times.
- **Baseline mismatch noted:** the expected baseline was 0 errors, 1 harmless "Get items" OData warning — but Warnings read (0) on every check that session.
- A `Set_varOutStatus` action was found to already contain `value: "OK"` unexplained — no edit had been made that session to produce it.
- After publishing (required to enable a Draft-status flow — see note below) and running a test, the run failed on a separate, already-known issue (Bug 5 — empty `sectionId` on one-off recapture).
- **Immediately afterward, with no field edit made by the user between checks**, Flow Checker went from 0 errors to **26 errors**, all `'Value' is required'`, across the same set of SetVariable/InitializeVariable actions (full list below). Screenshots captured.

**Side note on flow status**: this session also found the flow sitting in **Draft status, not Published**, and could not be enabled from the Workflows list ("Classic workflows can't be enabled from this list" — requires publishing from inside the flow itself). Not itself corruption, but worth mentioning as it affects how this flow must be re-enabled after any restore.

### Incident 5 — 15 August 2026, afternoon session, ~16:21
Occurred with **zero edits made** between a confirmed-clean check (0 errors, 1 warning, immediately following a successful isolated write-back of 4 actions) and this event. Flow Checker jumped to **21 errors**, concentrated in the recurring-branch and page-creation-branch actions — none of which had been touched in the session up to that point. The 4 just-fixed actions were not present in this error list, suggesting that specific fix had held.

### Incident 6 — 15 August 2026, shortly after Incident 5
After a single, correct, isolated edit to one action (`OF05b`), the error count **increased** (15 → 19) rather than decreased — the wrong direction for a correct isolated fix. A **new error type appeared for the first time**: `"Invalid parameters"` (distinct from `'Value' is required'`) on three untouched actions and on one not-yet-edited action. This is a different corruption signature than Incidents 1–5.

Despite this, continuing with further single-action isolated edits did **not** cause further escalation — the very next check showed a sharp drop (19 → 1), and shortly after, full recovery to 0 errors / 1 warning. This is inconsistent with a strict "any edit while corrupted makes it worse" rule, and suggests the corruption events may be partially decoupled from the specific edit immediately preceding them.

### Incident 4 addendum — publish-only validation gap (same session, after full 26-action recovery)
After successfully writing back all 26 corrupted values (verified via repeated isolated per-action Flow Checker checks, ending at 0 errors / 1 warning) and a clean `Save draft`, **`Publish` failed** with:

> `InvalidTemplate`: "The inputs of template action 'varTargetSectionPagesUrl' ... is invalid. Action 'Apply_to_each' must be a parent 'foreach' scope of action 'varTargetSectionPagesUrl' to be referenced by 'repeatItems' or 'items' functions."

Investigation found the flow's **top-level** `InitializeVariable` action for `varTargetSectionPagesUrl` (positioned early in the flow, structurally outside any `foreach` loop) had a `value` field set to `@items('Apply_to_each')?['pagesUrl']` — a cross-scope reference that is invalid at that action's position. Every other `InitializeVariable` action in this flow has no `value` field at all; this one action acquiring a stray, syntactically-plausible-but-structurally-invalid value (that happens to match a legitimate expression used correctly elsewhere, on a *different*, properly-scoped action later in the flow) is consistent with a corruption/cross-wiring event rather than user error. **Neither Flow Checker nor draft-save caught this — only Publish did.** Clearing the stray value (restoring the action to `name` + `type` only) resolved the issue; Publish then succeeded.

---

## Affected actions (26, Incident 4; substantially overlapping set in Incidents 5–6)

`varTargetSectionPagesUrl` (×2, InitializeVariable), `varOneNoteResolverResult` (×2, InitializeVariable), `Set varPageAction Created`, `Set varOutputPageSelfUrl Created`, `Set varOutputPageLink Created`, `Set varPageAction Created OneOff`, `Set varOutputPageSelfUrl Created OneOff`, `Set varOutputPageLink Created OneOff Gate`, `Set varPageAction ExistsNoCreate`, `Set varOutputPageSelfUrl Existing`, `Set varPageAction UpdatedAppend`, `Set varOutputPageLink Existing`, `Set varOutputPageLink Created OneOff`, `varFinalExistingPageSelfUrl` (InitializeVariable), `varFinalPageDecision` (InitializeVariable), `varFinalMatchCount` (InitializeVariable), `Set varTargetSectionPagesUrl OneOff Exists`, `Set varOneNoteResolverResult Exists OneOff`, `Set varTargetSectionPagesUrl OneOff Created`, `Set varOneNoteResolverResult Created OneOff`, `OF05a — Set varFinalExistingPageSelfUrl (OneOff)`, `OF05b — Set varFinalPageDecision (OneOff)`, `OF05c — Set varFinalMatchCount (OneOff)`, `Set varOutStatus`.

Notably, **the InitializeVariable actions in this list never had a `value` field in their original definition** (they only declare `name` and `type`) — so Flow Checker demanding a value for these specifically is itself part of the anomaly, not a normal validation state. The top-level `varTargetSectionPagesUrl` InitializeVariable (Incident 4 addendum) is the inverse case: it should also have no value field, but corruption gave it one.

---

## Request to Microsoft

1. Investigate whether a race condition exists in the Designer save pipeline between (a) a recently-set blank/empty value field not yet fully committed server-side, and (b) a concurrent structural save (inserting/editing another action) — while noting Incidents 5 and 6 occurred without an obvious qualifying edit, so this may not be the complete picture.
2. Confirm whether Flow Checker validation and draft-save validation are guaranteed to run the same checks as Publish — we have now observed disagreement in **both directions** (publish succeeding despite Flow Checker errors, and publish failing despite Flow Checker/draft-save showing clean).
3. Advise whether `InitializeVariable` actions without an explicit `value` field are expected to be flagged by Flow Checker as requiring a value, or whether this is itself a symptom of the corruption.
4. Investigate the "Invalid parameters" error variant seen in Incident 6 — is this a distinct corruption mechanism from the "Value is required" blanking pattern, or the same underlying issue surfacing differently?
5. Advise on a safe workflow for applying single-field edits to an existing flow without triggering this pattern — in particular whether a full solution export/import (editing `definition.json` outside the live Designer surface) avoids the trigger.

---

## Supporting evidence available on request

- Screenshots of the 26-error and 21-error Flow Checker states (15 August, multiple timestamps across the day).
- Screenshot of the "Invalid parameters" error variant (Incident 6).
- Screenshot of the publish-time `InvalidTemplate` error and its resolution.
- Full Peek Code capture of all affected actions, both pre- and post-corruption states, plus a full structural map of the entire flow (see `flow-reference-2026-08-15-full-peek-code-capture.md` in this folder).
- Prior handover docs for incidents 1–3 (listed above) and the two session-2 handovers for 15 August.
- Run history entry showing BadRequest on `Create_Page_OneOff` with empty `sectionId` (separate issue, Bug 5, confirmed to share root cause with this corruption pattern — `Create_Page_OneOff`'s `sectionId` parameter reads from a variable set exclusively by actions in the affected list above).

---

*Drafted 15 August 2026, updated same day (session 2, part 2) with Incidents 5–6 and the publish-only validation gap finding. Update this file with the actual Microsoft ticket number once submitted, and link it from the next session's handover.*
