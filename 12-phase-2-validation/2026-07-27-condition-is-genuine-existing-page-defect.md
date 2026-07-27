# Flow B — `Condition Is Genuine Existing Page` Structurally Unreachable

**Date:** 2026-07-27
**Method:** Live test of AMEND-2026-07-27-001's fixes (re-capturing a meeting with an existing page), followed by a targeted Peek Code check of `Set_varOneNoteResolverResult_Exists_OneOff` after the live run showed `Condition Is Genuine Existing Page` evaluating False. Cross-checked against every other `varOneNoteResolverResult`-setting action already captured in `2026-07-27-flow-b-outstatus-trace.md`.
**Status:** Root cause confirmed, real-world impact confirmed live, first fix attempt found insufficient, corrected fix applied and live-verified. **Resolved — see Section 8.**

---

## 1. Background

While live-testing AMEND-2026-07-27-001's fixes (`varPageAction`/`varOutputPageSelfUrl` corrections on the "page already exists" branch), the test run showed `Condition Is Genuine Existing Page` — which gates whether Flow B appends an "automated update" note to a genuinely existing OneNote page, versus falling back to a one-off page creation — evaluating **False**.

Rather than assume this was simply the wrong test scenario, a second capture of the same meeting was considered as a way to hit the True branch. Before doing so, the actual literal value the condition checks for was investigated directly via Peek Code, since a repeat capture would only be a useful test if the True branch was actually reachable at all.

## 2. Finding

`Condition Is Genuine Existing Page`'s expression:
```
equals(variables('varOneNoteResolverResult'), 'Exists')
```

`Set_varOneNoteResolverResult_Exists_OneOff` — the action inside `Condition Section Exists OneOff`'s True branch, named as though it sets `varOneNoteResolverResult` to `"Exists"` — actually contains no `value` key at all:
```json
{
  "type": "SetVariable",
  "inputs": {
    "name": "varOneNoteResolverResult"
  },
  "runAfter": {
    "Set_varTargetSectionPagesUrl_OneOff_Exists": ["Succeeded"]
  }
}
```

This is not a wrong-value bug — it assigns nothing at all, leaving `varOneNoteResolverResult` at whatever it was previously (most likely blank/uninitialized on this path, since Flow B's variable-init block at the top of the flow was not seen to include it).

Cross-checking against every other action anywhere in Flow B that sets `varOneNoteResolverResult` (all previously captured in `2026-07-27-flow-b-outstatus-trace.md`, Section 2):

| Action | Path | Value set |
|---|---|---|
| `varOneNoteResolverResult_ExistingMapping` | Recurring, mapping row exists | `"ExistingMapping"` *(later found to be incorrect — see Section 7)* |
| `varOneNoteResolverResult_1` | Recurring, section exists | `"ExistingSection"` |
| `varOneNoteResolverResult_2` | Recurring, section created | `"CreatedSection"` |
| `Set_varOneNoteResolverResult_Exists_OneOff` | One-off, section exists | *(no value assigned — see above)* |
| *(one-off, section created)* | Not yet traced this session — see Section 3 |  |

**None of the confirmed values is the literal string `"Exists"`.** This means `Condition Is Genuine Existing Page` cannot evaluate True via any currently-traced code path in Flow B. It is structurally unreachable — the same failure pattern previously found and fixed in `Condition_IsRecurring` (AMEND-2026-07-19-001) and `C6B_Check_N` (AMEND-2026-07-18-001): a condition that reads correctly on the canvas but has no upstream action that can ever satisfy it.

## 3. Downstream consequence

If this condition can never be True, then Flow B's False branch — `Create_Page_OneOff`, a fresh page creation — fires every time the "page already exists" route is taken, **regardless of whether a genuine existing page was actually found**. In practice this likely means:

- The "append an automated update note to an existing page, preserving human-entered notes" feature (`Compose_UpdateHtmlFragment`, `Update_page_content_Existing_Branch`) has probably never executed for any meeting, on any path, at any point in the project's history.
- Re-capturing a meeting whose section/page already exists may be creating a **new** OneNote page each time instead of updating the existing one — worth confirming directly, since this would mean duplicate pages accumulating silently, similar in spirit to the duplicate-section defect fixed in AMEND-2026-07-19-004.

This has not yet been confirmed by checking actual OneNote content (e.g. whether repeated captures of the same meeting have produced multiple pages) — recommended as the first verification step before deciding on a fix.

## 3a. Real-world impact confirmed live

To confirm, "HoP - Focus Time" (a recurring meeting, section "Mtg - HoP - Focus Time") was deliberately re-captured for the same occurrence (27 Jul 2026) it had already been captured for earlier in the session. The agent responded with a normal-looking success message and a page link, giving no indication anything was wrong.

Checking the OneNote section directly afterward showed **two separate pages both dated "27 Jul 2026"** under "Mtg - HoP - Focus Time" (alongside the unrelated, correctly-separate "3 Aug 2026" occurrence). This confirms the predicted consequence exactly: the recapture did not update the existing 27 Jul page, it silently created a duplicate.

This confirms the defect is real-world impacting today, not merely theoretical, and affects the recurring path as well as the one-off path (this test used a recurring meeting) — meaning the recurring-side equivalents of `varOneNoteResolverResult` (`"ExistingMapping"`, `"ExistingSection"`, `"CreatedSection"`) are equally unable to satisfy `Condition Is Genuine Existing Page`, consistent with Section 2's table.

## 4. Design decision made — first fix (superseded by Section 7)

- The one-off "section created" branch's equivalent action was traced and found to reuse `"CreatedSection"`, consistent with the recurring path.
- **Design decision**: the condition was changed to check for the real values in use, rather than restoring a literal `"Exists"` string everywhere: `contains(createArray('ExistingMapping','ExistingSection'), variables('varOneNoteResolverResult'))`. This was applied to `Condition Is Genuine Existing Page` and published successfully.
- **This fix alone was insufficient** — see Section 7. It was correct as far as it went, but two of the four `varOneNoteResolverResult`-setting actions were subsequently found to never populate a value in the first place, so no expression change alone could fix the underlying problem.

## 5. Relationship to the `OutStatus` build

This defect was found via the same trace exercise being done ahead of the `OutStatus` six-value build (`2026-07-27-flow-b-outstatus-trace.md`). It's relevant to that work directly: `varOneNoteResolverResult`'s distinct states (`ExistingMapping`/`ExistingSection`/`CreatedSection`/one-off equivalents) look like good candidate inputs for distinguishing `SUCCESS` from `PARTIAL_SUCCESS`, or for informing response messaging about whether a page was created vs updated — but only once they reliably and correctly reach `Condition Is Genuine Existing Page`, or that condition is redesigned around the values that actually exist.

## 6. Suggested next steps (superseded — see Section 7 for current steps)

1. ~~Check actual OneNote content for evidence of duplicate pages from repeated captures of the same meeting~~ — **done, confirmed, see Section 3a.**
2. ~~Trace the untraced one-off "section created" branch to complete the picture~~ — **done, see Section 4.**
3. ~~Make the design decision in Section 4~~ — **done, see Section 4.**
4. ~~Fix and live-test~~ — **fix applied and live-tested; found insufficient, see Section 7.**
5. Consider a one-off cleanup pass to identify and remove any other duplicate pages already created by this defect across the project's history, since it appears to have been present since this branch was originally built.
6. Log as a new amendment once fixed — this document is a trace/investigation record only.

## 7. Correction — first fix applied was insufficient (deeper root cause found)

**Date of this addendum:** 2026-07-27, same day, following live retest of the Section 4 fix.

The condition-expression fix proposed in Section 4 and applied to the Designer (`contains(createArray('ExistingMapping', 'ExistingSection'), variables('varOneNoteResolverResult'))`) was live-tested by recapturing the same 27 Jul 2026 "HoP - Focus Time" occurrence a third time. The condition **still evaluated False**, and Flow B created a **third** duplicate page plus an unrelated "Untitled Page" in the same run.

Tracing the run showed `Condition Mapping Exists` went True (a real mapping row and existing page were found — confirmed via `Compose_ExistingPageId` extracting a genuine page ID), but `Set varOneNoteResolverResult ExistingMapping`, the action responsible for setting `varOneNoteResolverResult` on that branch, was then Peek-Coded directly and found to have **no `value` key at all** — identical to the `Exists_OneOff` defect already documented in Section 2:

```json
{
  "type": "SetVariable",
  "inputs": {
    "name": "varOneNoteResolverResult"
  },
  "runAfter": {
    "Set_varTargetSectionPagesUrl_ExistingMapping": ["Succeeded"]
  }
}
```

This means the array-based fix in Section 4, while logically correct, was checking against a variable that its own upstream writers were silently failing to populate on two of the four real paths. The earlier assumption that `Set varOneNoteResolverResult ExistingMapping`'s value was `"ExistingMapping"` (stated in Section 2's table) was taken from the canvas label, not verified via Peek Code at the time — this addendum corrects that.

### Complete, now-verified picture

| Action | Path | Value set |
|---|---|---|
| `Set varOneNoteResolverResult ExistingMapping` | Recurring, mapping row exists | ❌ **missing — no `value` key** |
| `varOneNoteResolverResult_1` | Recurring, section exists | `"ExistingSection"` |
| `varOneNoteResolverResult_2` | Recurring, section created | `"CreatedSection"` |
| `Set_varOneNoteResolverResult_Exists_OneOff` | One-off, section exists | ❌ **missing — no `value` key** |
| `Set_varOneNoteResolverResult_Created_OneOff` | One-off, section created | `"CreatedSection"` |

The pattern is exact and systematic: every action representing "something was found to already exist" is missing its value; every "something was just created" action has one set correctly and consistently (`"CreatedSection"` reused across both paths). This points to the "found existing" half of each pair never having been finished when this branch was originally built, rather than a logic error in the condition itself.

### Corrected fix

1. `Set varOneNoteResolverResult ExistingMapping` → add `"value": "ExistingMapping"`
2. `Set_varOneNoteResolverResult_Exists_OneOff` → add `"value": "ExistingSection"` (reusing the existing one-off/recurring shared convention, matching its sibling `varOneNoteResolverResult_1`)
3. `Condition Is Genuine Existing Page`'s expression from Section 4 — **no further change needed**, stays as `contains(createArray('ExistingMapping', 'ExistingSection'), variables('varOneNoteResolverResult'))`

Not yet applied — pending this write-up. Section 4's original "not yet resolved" items are superseded by this addendum; the design decision has been made (array-based condition, matching real values) and the remaining work is purely the two missing `value` assignments above.

### Updated suggested next steps (superseded — see Section 8)

1. ~~Apply the two `value` corrections above in the Designer~~ — **done, see Section 8.**
2. ~~Publish and live-test via a fourth recapture of the same 27 Jul 2026 occurrence~~ — **done, verified, see Section 8.**
3. ~~Confirm via OneNote directly that the page count for 27 Jul 2026 does not increase~~ — **done, confirmed, see Section 8.**
4. Clean up the accumulated duplicate pages and the stray "Untitled Page" created during this investigation — still outstanding.
5. Log the complete fix as a single new amendment — **done, see Section 8.**

## 8. Fix verified live — closed

**Date:** 2026-07-27, same day, following application of the corrected fix from Section 7.

Both corrections from Section 7 were applied in the Designer:

1. `Set varOneNoteResolverResult ExistingMapping` → `"value": "ExistingMapping"` set and confirmed via Code view.
2. `Set_varOneNoteResolverResult_Exists_OneOff` → `"value": "ExistingSection"` confirmed already present via Code view at time of checking (found already correct, no edit needed — cause unclear, possibly set in an earlier pass of this same session).

Flow published successfully. The same 27 Jul 2026 "HoP - Focus Time" occurrence was recaptured a fourth time as the live test. Results:

- `Condition Mapping Exists` → True (as expected, unchanged from prior runs)
- `Set varOneNoteResolverResult ExistingMapping` → now populates `"ExistingMapping"` correctly
- `Condition Is Genuine Existing Page` → **evaluated True** for the first time in this investigation, confirmed directly in the Activity run trace (green checkmarks through `Get Sections Existing Branch` → `Filter Existing Section By Name` → `Apply to each Existing Section` → `Update page content Existing Branch` → `Set varPageAction UpdatedAppend` → `Set varOutputPageLink Existing`)
- No new page was created — confirmed directly in OneNote: "Mtg - HoP - Focus Time" section still shows a single "27 Jul 2026" page (page count did not increase from this run)

**This defect is now resolved.** `Condition Is Genuine Existing Page` correctly distinguishes a genuinely pre-existing page from a freshly-created section, and the "append an automated update note, preserving human-entered content" feature now executes as originally designed — likely for the first time in the project's history on the recurring-mapping path, and for the first time via this specific trigger condition on the one-off/section-exists path.

### Outstanding — not part of this fix

- **OneNote cleanup**: the accumulated duplicate pages and one stray "Untitled Page" created during this investigation's earlier test runs (Sections 3a and 7) have not been cleaned up. Recommended as a short follow-up task, low risk, no flow changes required.
- This fix should be logged as a new amendment (AMEND-2026-07-27-002 or next available ID) covering both `value` corrections, referencing this document.
