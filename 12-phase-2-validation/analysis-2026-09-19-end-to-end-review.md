# End-to-end review — Flows B, C, D — 19 Sep 2026

**Inputs:** `flow-reference-2026-09-19-flow-b-draft-post-wipe.md`, `flow-reference-2026-09-19-flow-c-published.md`, `flow-reference-2026-09-19-flow-d-published.md`, checked against `known-good-values-addendum-2026-09-18.md` and `session-2026-09-19-flow-c-restructure-and-reliability.md`.
**Flow A:** not re-captured; treated as the input contract to Flow B.
**Reviewer:** Claude (Opus 5), chat session 19 Sep 2026 evening.

---

## Restore status (Priority 1 in handover-2026-09-19-late-urgent.md)

- `Set_varOutStatus` in the saved draft matches the handover expression exactly; parens balance. **Resolved in draft.**
- Checked against the 18 Sep addendum and matching: `Set_varTargetSectionPagesUrl_ExistingMapping`, `varTargetSectionPagesUrl_1`, `varTargetSectionPagesUrl_2`, `Set_varTargetSectionPagesUrl_D2_Exists`, `Set_varOneNoteResolverResult_Exists_D2`, `Set_varTargetSectionPagesUrl_D2_Created`, `Set_varOneNoteResolverResult_Created_D2`, and all `Create_Mapping_Item_Recurring` fields including EndTime / JoinUrl.
- Still to do: Flow Checker 0 errors → publish → recurring test.

---

## High severity

### F1 — Flow C: AI recap call receives a literal string
`FC05g_Get_AI_Insight` → `aiInsightId` = `"outputs('FC05f_Best_Insight_Id')"` (no `@`). The ID sent is the literal text, so the call fails. FC06 runs after Failed, so chat capture still happens and the failure is masked.
**Fix:** `@outputs('FC05f_Best_Insight_Id')`.

### F2 — Flow D: offset is 1 minute
Captured `varOffsetMinutes` = `1`. Session log says 30; 18 Sep addendum says 5. Flow C sets `ChatCaptured = true` regardless of recap outcome, so D fires C ~1 min after the meeting ends, before the Copilot recap exists, then marks the row done. Recap is permanently lost even once F1 is fixed.
**Fix:** set to `30`.

### F3 — Flow B: recurring section logic runs for one-offs
`Condition_Mapping_Exists` False branch is not gated on IsRecurring. For a new one-off: one-off branch resolves `Evt Mtg - X`, then match count 0 → False branch looks up / creates `Mtg - X` and overwrites `varTargetSectionPagesUrl` (and sets resolver result `CreatedSection`). Page lands in a `Mtg -` section.
Likely source of the "D2 prefix still `Mtg -`" symptom.
**Verify:** in a one-off run history, check whether `Create_Section_Recurring` / `varTargetSectionPagesUrl_2` executed.
**Fix (expression-only, no action moves):**
- `Condition_Section_Exists_Recurring` → `@and(equals(toLower(string(triggerBody()?['text'])), 'true'), equals(outputs('Compose_Section_Match_Count_Recurring'), 1))` equals `true`
- `Condition_Section_Count_Is_Zero` → `@and(equals(toLower(string(triggerBody()?['text'])), 'true'), equals(outputs('Compose_Section_Match_Count_Recurring'), 0))` equals `true`

### F4 — Flow B: one-off existing-page path looks up the wrong section name (root cause of open bug 1)
`Condition_Is_Genuine_Existing_Page` re-finds the section by name via `Compose_SafeSectionName_ExistingBranch`, which builds `Mtg - ` names capped at 43 chars. One-off sections are `Evt Mtg - ` capped at 37. Filter returns nothing → `Apply_to_each_Existing_Section` runs zero times → no page created/updated, yet `varPageAction` = Updated → OutStatus SUCCESS.
A pre-loop page-count check will not fix this.
**Fix (expression-only):** `Filter_Existing_Section_By_Name` where → `@equals(item()?['pagesUrl'], variables('varTargetSectionPagesUrl'))`. `varTargetSectionPagesUrl` is already correct for both paths at this point (after F3). Name composes become unused; leave them in place for now.

---

## Medium severity

### F5 — Flow B: D2 branch is unreachable
`PAGE_EXISTS` needs a mapping row with PageSelfUrl; any match sends `Condition_Mapping_Exists` True, which always sets `ExistingMapping`. So `Condition_Is_Genuine_Existing_Page` is effectively always True. Also: `Condition_Section_Exists_D2` is `@true equals @true` (recovery placeholder); `Filter_OneNote_Section` / `Create_Section_D2` reference `Compose_SafeSectionName` not `_D2`; `Compose_SafeSectionName_D2` runs after the lookup and is unused.
Open bugs 2 and 3 in the handover are therefore cosmetic. Recommend leaving D2 untouched until the flow is stable (deletions have triggered every wipe), then removing it.

### F6 — Flow B: `Get_items` `$top 500` with no filter
Per-occurrence recurring rows mean the list will pass 500. Rows beyond the cap won't be returned → lookups miss → duplicate pages. Watch `OutSPItemCount`. Longer-term fix: OData `$filter` at source by SeriesMasterId + OccurrenceDate or MeetingId.

### F7 — Flow B: UJ3b deletes rows the filters still see
Filters read the pre-delete `Get_items` body. Add `not(empty(item()?['SectionPagesUrl']))` to `Filter_Existing_Mapping` and `OF01_—_Filter_Existing_Mapping_OneOff`.

### F8 — One-off meetings never get chat capture
FD04 requires SeriesMasterId; one-off mapping rows don't store OccurrenceDate / EndTime / JoinUrl. Design gap — pairs with the stored past/future-meeting idea.

---

## Drift from the 19 Sep session log

### F9 — Flow C: session 4 edits not present in published version
- `FC12` still includes `<hr><h2>Chat Transcript</h2>` (log says removed). `FC05s` also adds `<hr><h2>Meeting Capture</h2>`. With the new template both headings will duplicate.
- `FC05p` and `FC05q` `from` still start `@   outputs(` — same leading-space pattern fixed in FC05s.
- `FC02` has no `trim()` (18 Sep addendum had it). Low impact — Flow B trims JoinUrl on write.

---

## Hypothesis

### F10 — ContentValidationError in Teams
Likely the HTML Flow B returns to the agent rather than OneNote. `Create_OneNote_Page` wraps `text_3` in `<p>` (now nests `<h2>`/`<hr>`), and the Response returns the full page HTML twice (`outpagehtml`, `outupdatehtmlfragment`). Check whether any topic node renders either; if not, remove them from the Response (then refresh the topic's C10 bindings). Needs the exact error text or a topic trace to confirm.

---

## Minor notes
- `Set_varPageAction_UpdatedAppend` sets `Updated`, so `Compose_AgentResponseSummary`'s `UpdatedAppend` / `ExistsNoCreate` branches never match.
- Response `outbranchresult` returns `varFinalMatchCount`.
- `Filter_Pages_By_Title` calls `formatDateTime(text_5)` — fails if a one-off is called with empty OccurrenceDate.
- Title-set failure still blocks the mapping write (see finding-2026-08-16): if `Compose_ConfirmedCreatedPageId` is empty after the 5s delay, `Set_PageTitle_Recurring` fails and the OF09 gate is skipped.
- Mixed OneNote connections (`shared_onenote` / `shared_onenote-1`) and notebookKeys (`Documents/Meeting Notes` vs `Master Archive Folder/Meeting Notes`) — per session 4 the Master Archive key is deliberate; noted for consistency only.

---

## Recommended work order

Guiding rule: prefer expression edits over adding, moving or deleting actions — deletions have caused every wipe. Snapshot to GitHub before and after each flow change.

### Session 1 — restore and quick wins (Sonnet, lower effort)
1. Flow B: Flow Checker → 0 errors → Publish.
2. Flow B: run a known-good recurring capture. Confirm OutStatus SUCCESS and page created. Note whether ContentValidationError appears (baseline for F10).
3. Flow D: `varOffsetMinutes` → 30. Publish.
4. Flow C: F1 (`@` on aiInsightId), F9 (remove FC12 heading, strip leading spaces in FC05p/q; decide on FC05s heading). Publish.
5. Test C/D: set `ChatCaptured` = No on today's recurring test row; let D pick it up after the offset (or run C manually). Confirm recap + transcript appended once.
6. Update known-good values; snapshot C and D.

### Session 2 — Flow B one-off correctness (Opus)
7. Snapshot B.
8. F3: the two condition expression edits. Save.
9. F4: `Filter_Existing_Section_By_Name` where-clause edit. Save.
10. Test matrix: recurring new, recurring existing, one-off new, one-off existing, one-off with empty existing section. Check section names and that no `Mtg -` section is created for one-offs.
11. Publish; update known-good values; snapshot B.

### Session 3 — hardening
12. F7: add the empty-SectionPagesUrl guard to both mapping filters.
13. F10: confirm and, if right, remove HTML outputs from the Response and refresh topic bindings.
14. Minor: `Set_varPageAction_UpdatedAppend` value, `outbranchresult`, formatDateTime guard.

### Backlog
15. F6: source-side OData filter on Get_items (before list reaches ~400 rows).
16. F8 + past/future meeting idea: one-off chat capture (needs OccurrenceDate/EndTime/JoinUrl on one-off rows and FD04 change).
17. F5: remove D2 once the flow has been stable for a few sessions.

---
*Created 19 September 2026.*
