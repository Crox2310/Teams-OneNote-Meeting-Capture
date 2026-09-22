# Backlog — Teams → OneNote Meeting Capture

**Opened:** 22 Sep 2026, at the move from build to live field testing.
**Rule:** nothing here blocks field testing. New issues found in the field go here first, then get triaged. The system design is in `ARCHITECTURE.md`.

**Priority:** H = affects captures in normal use · M = affects edge cases or scale · L = tidy-up or enhancement
**Type:** Bug · Verify (repo and session notes disagree, or nobody has checked) · Improve · Tidy

---

## Open

| ID | Item | Type | Pri | Source | Notes |
|---|---|---|---|---|---|
| BL-01 | **FD06d skipped although FD06c took the True branch.** Seen 22 Sep on row 429 (EndTime `09/22/2026 09:30:00`), with the FD06d body intact in Code view. | Bug | H | Session 22 Sep | Possible causes: the offset hadn't elapsed yet, or the non-ISO EndTime format is misparsed by `ticks()`. Check FD06c's raw inputs in run history. |
| BL-02 | **Flow B JoinUrl source for one-off rows.** On 20 Sep `text_7` was removed from the trigger and the URL moved into `text_3`, but the 21 Sep D2 PostItem fix reads `text_7`. | Verify | H | Architecture draft | If it reads `text_7`, one-off rows get an empty JoinUrl and Flow C's FC05 fails for every one-off. Check the recurring Create Mapping Item action too. |
| BL-03 | **Live end-to-end test of a one-off meeting.** One-off support was built on 21 Sep but hasn't been run live yet. | Verify | H | Session 21 Sep | Depends on BL-02. May be covered by field testing. |
| BL-04 | **Flow D offset value.** The 19 Sep snapshot shows 1, while the notes and the 20 Sep handover say 30. | Verify | M | End-to-end review F2 | If it's 1, Flow C runs before the recap exists and the row is marked done, so the recap is lost. |
| BL-05 | **Flow C slot targets.** Confirm the recap and chat append into `#notes` / `#chat` rather than `body`, and whether the AllowFallback gate still exists. Check that FC12 and FC05s don't add duplicate headings (F9). | Verify | M | Weekend plan, review F9 | |
| BL-06 | **`Get_items` `$top 500` with no source filter in Flow B.** Once the list passes 500 rows, lookups miss and pages get duplicated. | Bug | M | Review F6 | Add an OData `$filter` by SeriesMasterId + OccurrenceDate, or by MeetingId. Watch `OutSPItemCount` and move this up once the list nears 350 rows. |
| BL-07 | **Stale skeleton rows (UJ3b).** A failed Flow B run leaves a row that blocks recapture, and the filters read the pre-delete data. | Bug | M | Review F7 | Add `not(empty(item()?['SectionPagesUrl']))` to `Filter_Existing_Mapping` and `OF01`. |
| BL-08 | **Title-set failure blocks the mapping write.** If `Set_PageTitle_Recurring` fails (intermittent 404), the OF09 gate is skipped and retries create duplicate pages. | Bug | M | Finding 16 Aug; review | Replace the 5-second delay with a Do-until poll. Consider running the mapping write first. |
| BL-09 | **Rename the section prefix `Mtg -` → `Rec -`** in both SafeSectionName composes and in OneNote. | Improve | L | Weekend plan C2 | Renaming keeps section IDs. Confirm whether this was already done. |
| BL-10 | **No re-fill after a late recap.** Once `ChatCaptured` is true, the occurrence is never revisited. | Improve | L | Architecture §9 | Options: a separate `RecapCaptured` flag, or a second pass at end + N hours for rows without a recap. |
| BL-11 | **Minor Flow B fixes:** `Set_varPageAction_UpdatedAppend` sets `Updated`; `outbranchresult` is bound to `varFinalMatchCount`; `Filter_Pages_By_Title` needs a guard for an empty `formatDateTime(text_5)`. | Bug | L | Review minor notes | Expression edits only. |
| BL-12 | **Flow A single-match path.** The Topic has no `MatchCount = 1` direct-confirm branch, so single-match days force a reselection prompt. | Verify | L | Session 6 Sep | May already be fixed. Check in the field. |
| BL-13 | **FA12B multi-match OnlineMeetingUrl** was hardcoded to `""`; the BodyPreview fallback only covers FA20C/FA29C. | Verify | M | Session 6 Sep | FA12B was edited on 18 Sep. Confirm whether multi-match picks now carry the join link. |
| BL-14 | **Remove dead paths:** Flow B D2 branch, `Compose_SafeSectionName_D2`, Flow A FA15–FA26 selection mode, the FA10–FA12 loop, the unused FA43 `endtime`. | Tidy | L | Review F5 | Only after several stable sessions, since deletions trigger value wipes. Snapshot first. |
| BL-15 | **Consistency:** mixed OneNote connections (`shared_onenote` / `-1`) and notebookKey paths. | Tidy | L | Review minor notes | Master Archive path is deliberate. Document it rather than change it unless it causes a fault. |
| BL-16 | **Clean-up:** delete the PA - Slot Test flow, test pages and empty "Meeting Capture" blocks; retire the unused `iCalUId` column. | Tidy | L | Weekend plan D5 | |
| BL-17 | **Known-good values:** merge the 18 Sep addendum into the canonical references and add Flow C and Flow D references. | Tidy | M | Weekend plan D4 | This is part of the write-up. |
| BL-18 | **C10 input order** (`text` / `text_1` / `text_2`) and the Copilot Studio tool links for A and B after the 20 Sep breakage. | Verify | L | Architecture draft | Working in the field suggests the links are fine. The input order still needs checking to document it. |
| BL-19 | **Page header content:** add attendees, organiser and start–end time. | Improve | L | Weekend plan decision 1 | |
| BL-20 | **Submit the Microsoft support ticket** covering the value-wipe corruption and Express mode self-reverting. | Improve | L | Ticket draft 15 Aug | The draft and discussion brief already exist. |

---

## Field-testing intake

Add issues found in the field below. Triage them into the table above at the end of each testing block.

| Date | Meeting type | What happened | Expected | Evidence (run ID, screenshot) |
|---|---|---|---|---|
| | | | | |

---

## Closed

| ID | Item | Closed | How |
|---|---|---|---|
| | | | |
