
# Flow B (PA - Resolve OneNote Meeting Section) — Full Structure Trace, ahead of OutStatus Build

**Date:** 2026-07-27
**Method:** Full peek-code trace of Flow B (Copilot Studio Designer → Code view, branch by branch), cross-checked against the Designer canvas, undertaken ahead of building the `OutStatus` six-value differentiation identified as the highest-leverage remaining item in `2026-07-20-gap-analysis-original-brief-vs-current-build.md`.
**Status:** Structure fully mapped. Two live defects found (see Section 3) and one gap in the trace (see Section 4), unrelated to OutStatus but discovered along the way. No fixes applied yet — this document is the map `OutStatus` values will be plotted against in a follow-up session.

---

## 1. Background

The 2026-07-20 gap analysis identified Flow B's `OutStatus` differentiation as the single highest-leverage remaining item: `Set_varOutStatus` is hardcoded to `"OK"` at one point, right before `Respond to the agent`, regardless of which branch actually executed. This is also the root fragility behind AMEND-2026-07-19-005 (the empty-string regression), since there is no structural protection — any accidental edit to that one Compose/SetVariable pair silently breaks every journey's status reporting.

Before writing any fix, this session traced Flow B's full branch structure via Peek Code, so the six target values (`SUCCESS`, `RECURRING_SETUP_REQUIRED`, `PARTIAL_SUCCESS`, `SETUP_SECTION_NOT_FOUND`, `SETUP_SECTION_AMBIGUOUS`, `ERROR`) can be mapped to real branch points rather than guessed at.

---

## 2. Full structure, as traced

```
Trigger (When an agent calls the flow)
→ Get items (SharePoint RecurringMeetingSectionMap)
→ init: varFinalExistingPageSelfUrl, varFinalPageDecision, varFinalMatchCount,
        varOutStatus, varOutputPageLink
→ Condition IsRecurring
    (uses equals(toLower(string(triggerBody()?['text'])), 'true') pattern —
     confirmed fixed per AMEND-2026-07-19-001)
```

**True — recurring path:**
```
Compose Input SeriesMasterId → Compose Input MeetingTitle → Filter Existing Mapping
→ Compose ExistingPageSelfUrl → Compose PageDecision → Compose Match Count
→ varFinalExistingPageSelfUrl1 / varFinalPageDecision1 / varFinalMatchCount1
→ Condition Mapping Exists
```
- **True** (row exists): Compose PageRouteExists → Compose Branch Result
  → Set varTargetSectionPagesUrl / varOneNoteResolverResult = `"ExistingMapping"`
- **False** (no row): Compose IgnoreSeriesMasterId → Compose PageRoute CreateRequired
  → Compose SectionDisplayName/SafeSectionName → Get/Filter/Match OneNote sections
  → **Condition Section Exists Recurring**
  - True: reuse section (`Apply to each` over filtered sections)
    → Set varTargetSectionPagesUrl / varOneNoteResolverResult = `"ExistingSection"`
  - False: `Create_Section_Recurring` (note: uses connection `shared_onenote-1`,
    inconsistent with `shared_onenote` used elsewhere — cosmetic, already flagged
    in the 20 July gap analysis)
    → Set varTargetSectionPagesUrl_2 / varOneNoteResolverResult_2 = `"CreatedSection"`
  → **Condition Should Write Mapping**
    (`equals(toLower(string(triggerBody()?['text'])), 'true')` pattern, confirmed fixed
    per AMEND-2026-07-19-002)
    - True: `Send_an_HTTP_request_to_SharePoint` (POSTs new mapping row: Title,
      SeriesMasterId, MeetingTitle, SectionPagesUrl, Status: "Active")
    - False: **confirmed genuinely empty** (`"else": {"actions": {}}`)

**False — one-off path:**
```
FB-F01 Compose Input MeetingTitle (one-off, empty-title-guarded per AMEND-2026-07-18-001)
→ Get/Filter/Match OneNote sections → Condition Section Exists OneOff
```
- True: reuse section (`For each 1`) → Set vars "...ExistsOneOff"
- False: `Create Section OneOff` → Set vars "...CreatedOneOff"

**Both paths merge →**
```
Condition Should Create Page
  (expression: equals(outputs('Compose_PageDecision'), 'PAGE_NOT_FOUND'))
```
- **True** (no existing page — create fresh):
  `Create_OneNote_Page` → `Compose_PageSelfUrl_Created`
  → `HTTP_Update_SP_PageSelfUrl` (writes PageSelfUrl back to the mapping row,
    resolving the item ID from either `Filter_Existing_Mapping` or the
    just-created `Send_an_HTTP_request_to_SharePoint` row)
  → `Set_varPageAction_Created` = `"Created"`
  → `Set_varOutputPageSelfUrl_Created`
  → `Set_varOutputPageLink_Created`

- **False** (page already exists — update in place):
  `Set_varPageAction_ExistsNoCreate` = `""` ⚠ **see Section 3**
  → `Set_varOutputPageSelfUrl_Existing` = `""` ⚠ **see Section 3**
  → `Compose_UpdateHtmlFragment` (builds an "Automated update" HTML note —
    "Updated by: Meeting Capture Agent... Existing human-entered notes were preserved.")
  → `Compose_ExistingPageId` (extracts page ID by splitting `varFinalExistingPageSelfUrl` on `/`)
  → **Condition Is Genuine Existing Page**
    (expression: `equals(variables('varOneNoteResolverResult'), 'Exists')`)
    - True: `Get_Sections_Existing_Branch` → `Filter_Existing_Section_By_Name`
      → `Apply_to_each_Existing_Section` → `Update_page_content_Existing_Branch`
        (appends `Compose_UpdateHtmlFragment`'s HTML to the existing page body)
      → `Set_varPageAction_UpdatedAppend` = `"Created"` ⚠ **mislabeled, see Section 3**
      → `Set_varOutputPageLink_Existing` = `outputs('Compose_ExistingPageSelfUrl')`
        ⚠ **undefined reference, see Section 4**
    - False: `Create_Page_OneOff` (connection `shared_onenote-1`)
      → `Set_varOutputPageLink_Created_OneOff`

**Then, regardless of branch:**
```
Compose AgentResponseSummary → Compose SP Item Count → Set varOutStatus → Respond to the agent
```

`Set varOutStatus` confirmed as a flat, unconditional value with no expression logic at all:
```json
{
  "type": "SetVariable",
  "inputs": { "name": "varOutStatus", "value": "OK" }
}
```

---

## 3. Live defects found (unrelated to OutStatus, discovered during this trace)

### Defect 1 — Two `SetVariable` actions show "Invalid parameters" in the Designer

`Set_varPageAction_ExistsNoCreate` and `Set_varOutputPageSelfUrl_Existing` (both inside `Condition Should Create Page`'s False branch) are both set to an empty string (`"value": ""`), and both currently display a red **"Invalid parameters"** badge on the canvas, with the Parameters tab reporting **"'Value' is required."**

This is a live, currently-published flow — the empty-string value clearly saved and ran successfully at some point, but the Designer's current validation no longer accepts it. This is a latent risk: if either action is ever opened, edited, and re-saved (or if a future change forces a full re-validation), publishing would likely be blocked until fixed.

**Recommendation:** give both a real fallback value rather than an empty string — e.g. `varPageAction` → `"Updated"` (see Defect 2, they're related), `varOutputPageSelfUrl` → reference `variables('varFinalExistingPageSelfUrl')` rather than blanking it.

### Defect 2 — `varPageAction` mislabeled as `"Created"` on the update-existing-page path

`Set_varPageAction_UpdatedAppend`, inside `Condition Is Genuine Existing Page`'s True branch (i.e. the path that **appends an update note to an existing page**, not creating anything), sets `varPageAction` to `"Created"`. This should almost certainly be `"Updated"` or similar — as written, downstream consumers of `varPageAction` (including the eventual `PARTIAL_SUCCESS`/`OutStatus` logic, and `varOutputPageSelfUrl` in Defect 1) cannot distinguish a genuine new-page creation from an existing-page update.

**Recommendation:** fix alongside Defect 1, since both live in the same branch and both feed into what `OutStatus`/response messaging will need to say accurately ("created a new page" vs "updated your existing page").

---

## 4. Gap in this trace — not yet resolved

`Set_varOutputPageLink_Existing` (same branch as Defect 2) references `outputs('Compose_ExistingPageSelfUrl')`. No action by this name was encountered anywhere in this session's trace of the recurring, one-off, or existing-page branches. Either:
- it exists somewhere not yet traced (the recurring "Condition Mapping Exists → True" branch's `Compose_Branch_Result`/`Compose_PageRoute_Exists` actions were seen on-canvas but not Peek-Coded — it may live there), or
- it's a genuinely broken/orphaned reference that would fail at runtime on this specific path (existing page + genuine `Exists` resolver result).

**Recommendation:** Peek Code `Compose_Branch_Result` and `Compose_PageRoute_Exists` (recurring, Mapping Exists = True branch) next session to confirm whether `Compose_ExistingPageSelfUrl` is defined there. If not found anywhere, this is a third live defect — worth a live test on the specific path (existing recurring mapping, genuine existing page) to confirm whether it actually fails.

---

## 5. Next steps

1. Peek Code `Compose_Branch_Result` / `Compose_PageRoute_Exists` to resolve the Section 4 gap.
2. Fix Defects 1 and 2 (related, same branch, both block clean re-validation and both affect message accuracy).
3. With the full structure now mapped, plan `OutStatus`'s six values against real branch points, e.g.:
   - `SUCCESS` — the `Create_OneNote_Page` True path, and the existing-page update True path (once Defect 2's mislabeling is fixed), both ending in `HTTP_Update_SP_PageSelfUrl` or the append-update succeeding.
   - `RECURRING_SETUP_REQUIRED` — `Condition Mapping Exists` False, surfaced back to the Topic rather than silently continuing (needs design decision: does this replace or supplement automatic section creation?).
   - `SETUP_SECTION_NOT_FOUND` / `SETUP_SECTION_AMBIGUOUS` — from `Compose_Section_Match_Count_Recurring` / `Compose_SectionMatchCountOneOff`, which currently only feed a True/False "exists" condition, not a distinct not-found-vs-ambiguous distinction.
   - `PARTIAL_SUCCESS` — OneNote write succeeds but `HTTP_Update_SP_PageSelfUrl` or `Send_an_HTTP_request_to_SharePoint` fails; currently no failure handling exists on any of these actions at all (no "configure run after" set up anywhere seen in this trace).
   - `ERROR` — needs "configure run after" added to the OneNote and SharePoint actions throughout, since a failed action currently fails the whole flow rather than surfacing a status, per the original gap analysis.
4. Backfill this session as a new amendment-log entry once fixes are actually applied (per the project's controlled-amendment process) — this document alone is a trace/investigation record, not yet an amendment.
