
# Backlog — Teams → OneNote Meeting Capture

**Opened:** 22 Sep 2026, at the move from build to live field testing.
**Last updated:** 23 Sep 2026 — all questions from end-to-end review answered; see `12-phase-2-validation/analysis-2026-09-22-end-to-end-review-DRAFT.md` (final).
**Rule:** nothing here blocks field testing. New issues found in the field go here first, then get triaged. The system design is in `ARCHITECTURE.md`.

**Priority:** H = affects captures in normal use · M = affects edge cases or scale · L = tidy-up or enhancement
**Type:** Bug · Verify · Improve · Tidy

---

## Open

| ID | Item | Type | Pri | Source | Notes |
|---|---|---|---|---|---|
| BL-36 | **One-off capture creates two sections: `Evt Mtg -` and `Mtg -`.** The `Evt Mtg -` prefix path was partially built but the old `Mtg -` code path was never removed. Both fire on every one-off capture, creating duplicate sections in OneNote. | Bug | H | Q2 answer 23 Sep | Struct/Expr. Requires Opus design session — need to gate/remove the `Mtg -` path and ensure `Evt Mtg -` is the sole prefix. Do not touch until Session 2. |
| BL-22 | **Recap duplicated when chat step fails.** FC16 (`Update_Item_ChatCaptured`) currently runs after FC15: Succeeded only. But FC05t has already appended the recap — so if FC15 fails, the next retry re-appends the recap with no chat content. **Q6 decision: `ChatCaptured` should only be true when content was actually written.** FC16 must require both FC05t (recap) and FC15 (chat) to have Succeeded. Accept duplicate recaps on retry rather than marking captured with missing content. | Bug | H | R-02 / Q6 answer | Settings. FC16 run-after: require FC05t Succeeded AND FC15 Succeeded. |
| BL-24 | **Recurring section logic runs for one-offs.** `Condition_Section_Exists_Recurring` and `Condition_Section_Count_Is_Zero` have no IsRecurring gate. | Bug | H | F3 / R-04 | Expr. Add IsRecurring gate to both conditions. Part of Session 2 — do alongside BL-36. |
| BL-25 | **One-off re-capture reports SUCCESS falsely.** `Filter_Existing_Section_By_Name` uses a 43-char name match — nothing is written but the user is told it saved. | Bug | H | F4 / R-05 | Expr. Filter by `pagesUrl` instead. Part of Session 2. |
| BL-03 | **Live one-off test matrix (full).** Set-up and auto-fill confirmed working for rows 7–9. Full matrix (new, re-capture, long title, `Evt Mtg -` prefix) still needed after BL-36, BL-24, BL-25 are fixed. | Verify | H | Session 21 Sep | After Session 2 fixes. |
| BL-26 | **Existing page found by date substring.** `Filter_Pages_By_Title` matches on a date text substring; fallback creates an untitled page with no mapping update. | Bug | M | R-06 | Expr. Match by page id. Part of Session 2. |
| BL-28 | **`/meet/` join links produce empty JoinUrl → poison rows.** FA29D and FA12B only extract `/l/meetup-join/` format. Blocked by FD04 guards but never filled. No current meetings affected (Q4 confirmed 23 Sep) — monitor for new meeting invites. | Bug | M | R-09 | Expr. Handle both formats. Scratch Diagnostics proof first. Reprioritise to H if a `/meet/` meeting is found in the field. |
| BL-29 | **FC01 can re-select an already-captured row.** `$top 1` with no `ChatCaptured eq 0` guard. | Bug | M | R-10 | Expr. Add `and ChatCaptured eq 0` to FC01 `$filter`. Preventive — no duplicates currently. |
| BL-07 | **Stale skeleton rows block recapture (UJ3b).** A failed Flow B run leaves a row with blank `SectionPagesUrl`; subsequent captures take the Mapping Exists branch and fail. | Bug | M | F7 | Expr. Add `not(empty(item()?['SectionPagesUrl']))` guard to `Filter_Existing_Mapping` and `OF01`. |
| BL-08 | **Title-set failure blocks the mapping write.** If `Set_PageTitle_Recurring` fails (intermittent 404), the OF09 gate is skipped and retries create duplicate pages. | Bug | M | Finding 16 Aug | Struct. Replace 5-second delay with Do-until poll. |
| BL-30 | **Flow B hard failures return nothing to the Topic.** No Scope/catch — Topic gets generic error, C12 retry unreachable, retries can create duplicate pages. | Bug | M | R-11 | Struct. Add Scope + catch. |
| BL-32 | **Rewrite drifted `ARCHITECTURE.md` sections.** Confirmed drift: Flow A uses V3 connector not V4; FD03 has no date filter (never built); Flow C appends to `body`; C10 contract (`text_3` HTML skeleton, `text_7` = JoinUrl); one-off prefix is `Evt Mtg -` per-title, not a fixed section. | Tidy | M | R-08 / R-17 | Docs. Do in Session 3 after Session 2 fixes are stable. |
| BL-17 | **Known-good values: merge the 18 Sep addendum and add Flow C and D references.** | Tidy | M | Weekend plan D4 | Part of the wider write-up. Session 3. |
| BL-06 | **`Get_items` `$top 500` in Flow B, no source filter.** Past 500 rows, lookups miss. | Bug | L | F6 | Expr. Not urgent — 9 rows. Move up when list nears 350. |
| BL-23 | **FD03 `$top 50`, no filter or order.** Past 50 rows the scheduler misses new meetings. | Bug | L | R-03 | Expr. `ChatCaptured eq 0` filter + `ID desc` order. Not urgent — 9 rows. Apply before list reaches 40. |
| BL-09 | **Rename one-off section prefix `Mtg -` → `Evt Mtg -` consistently.** The intended prefix is `Evt Mtg -` (Q2 confirmed). The duplicate section bug (BL-36) must be fixed first. | Improve | L | Q2 answer / weekend plan C2 | After BL-36 in Session 2. |
| BL-10 | **No re-fill after a late recap.** Once `ChatCaptured` is true the occurrence is never revisited. | Improve | L | Architecture §9 | Options: separate `RecapCaptured` flag, or second pass at EndTime + N hours. |
| BL-11 | **`outbranchresult` bound to `varFinalMatchCount`.** Should coalesce branch results. | Bug | L | Review minor | Expr. Session 2. |
| BL-12 | **Flow A single-match path forces reselection.** Prompts user to type "1" on single-match days. Confirmed acceptable in practice (Q8). | Improve | L | Session 6 Sep | Leave at L. |
| BL-31 | **Chat capped at 50 messages, no paging.** Busy meetings truncated. | Improve | L | R-12 | Document limit for now. |
| BL-33 | **UJ3b delete failure blocks Flow B.** One delete fail = whole capture fails. | Bug | L | R-13 | Settings. Add Failed to run-after. |
| BL-34 | **Re-capture appends full skeleton with misleading text.** | Bug | L | R-14 | Expr. Replace with header + datestamp. Session 2. |
| BL-35 | **"Meeting Invite" heading always empty.** BodyPreview resolved in Flow A but never passed to Flow B. | Improve | L | R-08 | Topic expression. Low priority. |
| BL-14 | **Remove dead paths** (expanded): Flow B D2 branch; `Compose_SafeSectionName_D2`; FA15–FA26; FA10–FA12 loop; unused FA43 `endtime`; `Compose_IgnoreSeriesMasterId`; unbound `outpagehtml`/`outupdatehtmlfragment`; Topic C9B `PageTitle`; Flow C `AllowFallback`. | Tidy | L | F5 / R-16 | Struct. Only after several stable sessions; snapshot first. |
| BL-15 | **Mixed OneNote connections and notebookKey paths.** Master Archive path is deliberate. | Tidy | L | Review minor | Document rather than change. |
| BL-16 | **Clean-up:** delete PA - Slot Test flow, test pages; retire unused `iCalUId` column. | Tidy | L | Weekend plan D5 | |
| BL-19 | **Page header content:** add attendees, organiser and start–end time. | Improve | L | Weekend plan | |
| BL-20 | **Submit the Microsoft support ticket** (value-wipe corruption, Express mode). | Improve | L | Ticket draft 15 Aug | Draft exists. |
| BL-37 | **Run history shows Failed for expected/handled outcomes (e.g. missing file/data).** The flow's overall Activity status inherits Failed from any internal action that returned Failed, even when a downstream branch handles it gracefully (e.g. file/data not available is an expected case, not a real error). David wants runs to read Succeeded when the outcome was handled as expected, and Failed reserved for genuine errors. | Improve | L | Chat 25 Sep | Fix: Configure run after on the handling branch ("is successful" + "has failed"), then add a Terminate action at the end of that branch set to Status: Succeeded so the run's overall status reflects the actual outcome. Not yet applied to any specific flow/action — needs a target chosen when picked up. |
| BL-38 | **Flow A candidate list should show a captured indicator per meeting.** When Flow A lists available meetings to select from, David wants each entry flagged if it's already been captured (page/notes via Flow B, chat/recap via Flow C+D), so he can see capture status in Agent chat without checking Copilot Studio or OneNote directly. | Improve | L | Chat 25 Sep | Design: add a `Get items` on `RecurringMeetingSectionMap` (date-window filtered, like Flow D's) before candidate-list composition; match each candidate by SeriesMasterId+OccurrenceDate (recurring) or MeetingId+OccurrenceDate (one-off); derive "notes captured" (row exists, `SectionPagesUrl` populated) and "chat/recap captured" (`ChatCaptured` = true); append an indicator to each line in `FA32_Compose_OutCandidateList*` (single/multi-match and one-off branches). Touches all three candidate-composition branches — moderate structural change, no Opus design session needed. |

---

## Field-testing intake

Add issues found in the field below. Triage into the table above at the end of each testing block.

| Date | Meeting type | What happened | Expected | Evidence (run ID, screenshot) |
|---|---|---|---|---|
| | | | | |

---

## Closed

| ID | Item | Closed | How |
|---|---|---|---|
| BL-01 | FD06d skipped although FD06c took the True branch | 23 Sep 2026 | Superseded — FD06d fires correctly. Root cause was empty JoinUrl on row 2. Row 2 deleted 23 Sep. |
| BL-02 | Flow B JoinUrl source for one-off rows | 23 Sep 2026 | Both Create Mapping Item actions confirmed to read `text_7`; published Topic YAML confirmed `text_7` = JoinUrl. |
| BL-04 | Flow D offset value | 23 Sep 2026 | FD01 = 30 confirmed in Code view. |
| BL-05 | Flow C slot targets | 23 Sep 2026 | Superseded — Flow C appends to `body`; no `#notes`/`#chat` slots; AllowFallback gate not present. Docs need updating (BL-32). |
| BL-12 | Flow A single-match forces reselection | 23 Sep 2026 | Confirmed acceptable in practice (Q8). Stays in open at L as an Improve item. |
| BL-13 | FA12B multi-match OnlineMeetingUrl hardcoded | 23 Sep 2026 | Fixed 18 Sep — FA12B extracts `/l/meetup-join/` href. `/meet/` format tracked as BL-28. |
| BL-18 | C10 input order and tool links | 23 Sep 2026 | Published Topic YAML confirmed: `text` = IsRecurring, `text_1` = Title, `text_2` = SeriesMasterId, `text_7` = JoinUrl. |
| BL-27 | EndTime US-format text — verify `ticks()` parsing | 23 Sep 2026 | Scratch Diagnostics confirmed: `ticks('09/22/2026 09:30:00')` = `ticks('2026-09-22T09:30:00Z')` = 639,256,662,000,000,000. No normalisation fix needed. |
