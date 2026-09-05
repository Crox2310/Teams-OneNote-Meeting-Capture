
# Session handover — 5 September 2026 — Corruption recurrence and new one-off D2 BadRequest defect

**Session context:** Live test of the OneNote Meeting Capture agent (Logistics Network - Deliverability Workshop, recurring series) surfaced a chain of issues. None resulted in a fully successful end-to-end capture by end of session.

---

## Part 1 — Corruption recurrence (three incidents, one session)

### Trigger

Live Teams test against a recurring meeting returned the generic error message. Tracing `Topic.OutStatus` back through Flow B showed `Set_varOutStatus` (the flow's final action) had lost its value expression — consistent with the known platform corruption pattern documented in `microsoft-discussion-brief-corruption-bug.md`.

### Incident timeline

1. **Initial state:** cross-checking the full Flow B peek code against `known-good-values-master-reference.md` found ~24 blanked `SetVariable` actions across the recurring, one-off, and Stage 1 safety-net branches, plus `Set_varOutStatus` itself. Restored in stages against the reference doc.
2. **First correction needed:** `Set_varOutputPageLink_Created_OneOff` was initially given the `_OneOff_Gate` sibling's value (referencing `Create_OneNote_Page`). This failed on save with a structural `runAfter` validation error, because `Create_Page_OneOff` — not `Create_OneNote_Page` — is the actual create action in that branch. Corrected to `@outputs('Create_Page_OneOff')?['body']?['links']?['oneNoteWebUrl']?['href']`.
3. **Second incident (same session):** after a clean save, Flow Checker reported 4 new blanked actions not seen in the first pass (`varTargetSectionPagesUrl_1`/`_2`, `varOneNoteResolverResult_1`/`_2`). Restored from reference.
4. **Third incident (same session):** Flow Checker then reported 33 errors, including several **already restored earlier in this same session**. Confirmed as real (not a stale panel) via direct Peek Code check on `Set_varOutStatus` — the `value` field was present in the JSON but set to a literal empty string (`"value": ""`), rather than the key being removed entirely.

### New corruption variant identified

The existing brief documents two patterns: key removed entirely, and character-level truncation (the `OF05c` `tring(...)` incident). **Tonight adds a third: key retained, value set to `""`.** Worth adding to `microsoft-discussion-brief-corruption-bug.md`'s technical detail section.

### Escalation pattern

Within a single session, with no structural edits between checks — only value restoration and saves — the error count moved **~24 → 4 → 33 → 32 (after full restoration) → 0**. This is a materially stronger data point for the Microsoft brief than anything currently logged: same-session recurrence, including re-corruption of already-restored actions, following only value-paste-and-save actions (no canvas restructuring).

### Unresolved: possible Designer load-timing artifact

Observed that reopening the flow shows action values as empty for a few seconds before they populate. Raised as an alternative (or contributing) explanation for at least some of tonight's Flow Checker/Peek Code reads — if a check fires during that load window, it could report a false blank. A clean test (reopen flow, wait 20-30s untouched, then check Peek Code and Flow Checker before any edit) was proposed but **not completed** this session. Does not explain the original live-test failure (`OutStatus` never reaching `SUCCESS`), which is a runtime result, not a Designer snapshot.

### Documentation gap: two undocumented actions

`Set_varPageAction_Created_D2` and `Set_varOutputPageSelfUrl_Created_D2` are not present in `known-good-values-master-reference.md` (they postdate the 31 Aug "last verified" date). Restored this session by inference from the equivalent `_OneOff` pattern, **not verified against a working run**:

| Action | Value used (unconfirmed) |
|---|---|
| `Set_varPageAction_Created_D2` | `Created` (literal) |
| `Set_varOutputPageSelfUrl_Created_D2` | `@body('Create_Page_OneOff')?['self']` |

**Do not treat these as known-good until confirmed against a successful run** — see Part 2, which suggests the branch containing these actions may not have executed successfully yet at all.

### End state, Part 1

All identified blanked actions restored. Flow Checker: 0 errors. Published successfully. Correction log entry needed once resolved (see Part 3).

---

## Part 2 — New defect: BadRequest in `Create_Page_OneOff` (sectionId empty)

Live test after the corruption restoration failed with a **different, non-corruption error**:

> `Flow run failed. Action 'Create_Page_OneOff' failed: The section id given in the input is invalid. If a custom value was entered, please try selecting from the supplied values.`

Run history shows `sectionId` was blank in the actual request. Confirmed this fired inside the **false branch of `Condition_Is_Genuine_Existing_Page`** — the "not a genuine existing page → fall back to one-off create" path, which contains `Create_Page_OneOff`, `Compose_SafePageTitle_OneOff`, and the two undocumented D2 actions above.

### Working hypothesis

This branch postdates the last fully-verified reference snapshot (31 Aug) and may **never have been exercised end-to-end on a live run before tonight**. If so, this isn't corruption or a restoration mistake — it's a genuine logic gap: nothing in this specific fallback path re-resolves `varTargetSectionPagesUrl` before `Create_Page_OneOff` consumes it, unlike the main one-off branch (`OF01`-`OF05` chain), which does resolve/create the section first.

### Not yet done — next session should start here

- Check `variables('varTargetSectionPagesUrl')`'s actual value at the point of failure in this run's history: genuinely never set (design gap in this branch) vs. set upstream and lost (corruption again). This determines which fix is needed.
- If it's a design gap: this D2 fallback branch likely needs its own section-resolve-or-create sequence (mirroring the `Get_Sections_OneOff` / `Filter_OneNote_Section_OneOff` / `Create_Section_OneOff` pattern) before calling `Create_Page_OneOff`, rather than assuming `varTargetSectionPagesUrl` is already populated.
- Once fixed and confirmed working, add the two D2 actions to `known-good-values-master-reference.md` for real (not inferred).

---

## Part 3 — Outstanding actions, in priority order

1. **Diagnose the D2 branch BadRequest** (Part 2) — check the actual value of `varTargetSectionPagesUrl` in run history before writing any fix.
2. **Confirm and document the two D2 actions** in `known-good-values-master-reference.md` once the above is resolved and a live run succeeds through that branch.
3. **Add tonight's corruption incidents to `microsoft-discussion-brief-corruption-bug.md`**: the same-session escalation pattern (including re-corruption of already-restored actions) and the new empty-string-value variant.
4. **Complete the Designer load-timing test** (reopen, wait 20-30s untouched, check Peek Code/Flow Checker) to determine whether it's contributing to false corruption reads — separate this from genuine corruption before it further complicates future incident logs.
5. **File the Microsoft support ticket** — still overdue per `CURRENT-STATE.md` process debt, and now has stronger supporting evidence from tonight.
6. **Re-test the Logistics Network capture end-to-end** once the D2 branch defect is fixed — no successful capture was completed this session.

---

## Status at session close

- Flow B published with all identified corruption restored; Flow Checker 0 errors.
- Live capture **still not successful** — blocked on the `Create_Page_OneOff` BadRequest in the one-off fallback branch (Part 2), which is a separate, apparently-genuine defect rather than corruption.
- No changes made to `known-good-values-master-reference.md` itself this session — pending confirmation per item 2 above.
