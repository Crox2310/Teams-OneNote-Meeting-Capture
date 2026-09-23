# Backlog — Teams → OneNote Meeting Capture

**Opened:** 22 Sep 2026, at the move from build to live field testing.
**Last updated:** 23 Sep 2026 — major triage following end-to-end review (see `12-phase-2-validation/analysis-2026-09-22-end-to-end-review-DRAFT.md`).
**Rule:** nothing here blocks field testing. New issues found in the field go here first, then get triaged. The system design is in `ARCHITECTURE.md`.

**Priority:** H = affects captures in normal use · M = affects edge cases or scale · L = tidy-up or enhancement
**Type:** Bug · Verify · Improve · Tidy

---

## Open

| ID | Item | Type | Pri | Source | Notes |
|---|---|---|---|---|---|
| BL-22 | **Recap duplicated when chat step fails.** FC16 runs only if FC15 succeeds, but FC05t already appended the recap — so every retry cycle re-appends the recap. | Bug | H | R-02 | Settings fix: FC16 run-after on Succeeded + Failed + Skipped + TimedOut. Decide Q6 (mark captured anyway, or keep retrying?) first. |
| BL-24 | **Recurring section logic runs for one-offs (F3).** `Condition_Section_Exists_Recurring` and `Condition_Section_Count_Is_Zero` have no IsRecurring gate. Long one-off titles create two sections. | Bug | H | F3 / R-04 | Expr. Fix depends on Q2 (one-off section design). |
| BL-25 | **One-off re-capture reports SUCCESS falsely (F4).** `Filter_Existing_Section_By_Name` uses a 43-char name match — nothing is written but the user is told it saved. | Bug | H | F4 / R-05 | Expr. Filter by `pagesUrl` instead. |
| BL-03 | **Live one-off end-to-end test — full matrix.** Set-up and auto-fill confirmed working for rows 7–9 (one-offs with a valid JoinUrl). Full matrix (new, re-capture, long title) still needed. | Verify | H | Session 21 Sep | Depends on BL-24, BL-25 being fixed first for re-capture cases. |
| BL-26 | **Existing page found by date substring.** `Filter_Pages_By_Title` matches on a date text substring; fallback creates an untitled page with no mapping update. | Bug | M | R-06 | Expr. Match by page id instead. Fix `Guard_Create_Page_Fallback` too. |
| BL-28 | **`/meet/` join links produce empty JoinUrl → poison rows.** FA29D and FA12B only extract `/l/meetup-join/` format. Any meeting whose invite body contains only a `/meet/` link gets an empty JoinUrl. Blocked by the new FD04 guards but never filled. | Bug | M | R-09 | Expr. Handle both formats in FA12B and FA29D–F. Prove `GetOnlineMeeting` accepts `/meet/` links in Scratch Diagnostics first. Answer Q4 (prevalence) first. |
| BL-29 | **FC01 can re-select an already-captured row.** `$top 1` with no `ChatCaptured eq 0` guard — if a duplicate row exists, FC01 can pick a captured row. | Bug | M | R-10 | Expr. Add `and ChatCaptured eq 0` to FC01 `$filter`. Preventive — no duplicates currently. |
| BL-07 | **Stale skeleton rows block recapture (UJ3b).** A failed Flow B run leaves a row with blank `SectionPagesUrl`; subsequent captures take the Mapping Exists branch and fail. | Bug | M | F7 | Expr. Add `not(empty(item()?['SectionPagesUrl']))` guard to `Filter_Existing_Mapping` and `OF01`. |
| BL-08 | **Title-set failure blocks the mapping write.** If `Set_PageTitle_Recurring` fails (intermittent 404), the OF09 gate is skipped and retries create duplicate pages. | Bug | M | Finding 16 Aug | Struct. Replace the 5-second delay with a Do-until poll. Consider running mapping write first. |
| BL-30 | **Flow B hard failures return nothing to the Topic.** No Scope/catch — the Topic gets a generic error, C12 retry is unreachable, and retries can create duplicate pages. | Bug | M | R-11 | Struct. Add Scope + catch. |
| BL-32 | **Rewrite drifted `ARCHITECTURE.md` sections.** Confirmed drift: Flow A uses V3 connector not V4; FD03 has no date filter (never built); Flow C appends to `body`; C10 contract (`text_3` HTML skeleton, `text_7` = JoinUrl). One-off section design pending Q2. | Tidy | M | R-08 / R-17 | Docs. Answer Q2 first for the one-off section description. |
| BL-17 | **Known-good values: merge the 18 Sep addendum and add Flow C and D references.** | Tidy | M | Weekend plan D4 | Part of the wider write-up. |
| BL-06 | **`Get_items` `$top 500` in Flow B, no source filter.** Past 500 rows, lookups miss. | Bug | L | F6 | Expr. Add OData `$filter`. Not urgent — 9 rows currently. Move up when list nears 350. |
| BL-23 | **FD03 `$top 50`, no filter or order.** Past 50 rows the scheduler only sees the oldest 50 and misses new meetings. | Bug | L | R-03 | Expr. Add `ChatCaptured eq 0` filter and `ID desc` order. Not urgent — 9 rows currently. Apply before list reaches 40. |
| BL-09 | **Rename section prefix `Mtg -` → `Rec -`** in both SafeSectionName composes and in OneNote. | Improve | L | Weekend plan C2 | Renaming keeps section IDs. Answer Q2 first. |
| BL-10 | **No re-fill after a late recap.** Once `ChatCaptured` is true the occurrence is never revisited even if the recap arrived late. | Improve | L | Architecture §9 | Options: separate `RecapCaptured` flag, or second pass at EndTime + N hours. |
| BL-11 | **Minor Flow B fixes:** `outbranchresult` is bound to `varFinalMatchCount` (should coalesce branch results); `Filter_Pages_By_Title` needs a guard for empty `formatDateTime(text_5)`. | Bug | L | Review minor | Expr. |
| BL-12 | **Flow A single-match path forces reselection.** Topic has no `MatchCount = 1` direct-confirm branch; single-match days prompt the user to type "1". | Improve | L | Session 6 Sep | May be acceptable in practice — answer Q8 from field use. |
| BL-31 | **Chat capped at 50 messages, no paging.** Recurring meetings share one thread; busy meetings are truncated. | Improve | L | R-12 | Struct. Document limit for now. |
| BL-33 | **UJ3b delete failure blocks Flow B.** `IsRecurring` runs after `UJ3b_Delete_Stale_Rows: SUCCEEDED` only — one failed delete fails the whole capture. | Bug | L | R-13 | Settings. Add Failed to run-after. |
| BL-34 | **Re-capture appends full skeleton with misleading "preserved below" text.** | Bug | L | R-14 | Expr. Replace with a single header + datestamp. |
| BL-35 | **"Meeting Invite" heading always empty.** BodyPreview is resolved in Flow A but never passed to Flow B via `text_3`. | Improve | L | R-08 | Topic expression. Low priority. |
| BL-14 | **Remove dead paths** (expanded): Flow B D2 branch; `Compose_SafeSectionName_D2`; Flow A FA15–FA26 selection mode; FA10–FA12 loop; unused FA43 `endtime`; `Compose_IgnoreSeriesMasterId`; unbound `outpagehtml`/`outupdatehtmlfragment`; Topic C9B `PageTitle`; Flow C `AllowFallback`. | Tidy | L | F5 / R-16 | Struct. Only after several stable sessions; snapshot first. |
| BL-15 | **Consistency:** mixed OneNote connections (`shared_onenote` / `-1`) and notebookKey paths. | Tidy | L | Review minor | Master Archive path is deliberate. Document rather than change unless it causes a fault. |
| BL-16 | **Clean-up:** delete PA - Slot Test flow, test pages, empty "Meeting Capture" blocks; retire unused `iCalUId` column. | Tidy | L | Weekend plan D5 | |
| BL-19 | **Page header content:** add attendees, organiser and start–end time. | Improve | L | Weekend plan | |
| BL-20 | **Submit the Microsoft support ticket** covering value-wipe corruption and Express mode self-reverting. | Improve | L | Ticket draft 15 Aug | Draft and discussion brief already exist. |

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
| BL-01 | FD06d skipped although FD06c took the True branch | 23 Sep 2026 | Superseded — FD06d fires correctly. Root cause was empty JoinUrl on row 2 (Winning Together Peak call). Row 2 deleted 23 Sep. |
| BL-02 | Flow B JoinUrl source for one-off rows | 23 Sep 2026 | Both Create Mapping Item actions confirmed to read `text_7`; published Topic YAML confirmed `text_7` = JoinUrl. |
| BL-04 | Flow D offset value | 23 Sep 2026 | FD01 = 30 confirmed in Code view. |
| BL-05 | Flow C slot targets | 23 Sep 2026 | Superseded — Flow C appends to `body`; no `#notes`/`#chat` slots exist in the current build; AllowFallback gate not present. Docs need updating (BL-32). |
| BL-13 | FA12B multi-match OnlineMeetingUrl hardcoded | 23 Sep 2026 | Fixed on 18 Sep — FA12B now extracts `/l/meetup-join/` href from the invite body. `/meet/` format tracked separately as BL-28. |
| BL-18 | C10 input order and tool links | 23 Sep 2026 | Published Topic YAML confirmed: `text` = IsRecurring, `text_1` = Title, `text_2` = SeriesMasterId, `text_7` = JoinUrl. |
| BL-27 | EndTime US-format text — verify `ticks()` parsing | 23 Sep 2026 | Scratch Diagnostics confirmed: `ticks('09/22/2026 09:30:00')` = `ticks('2026-09-22T09:30:00Z')` = 639,256,662,000,000,000. Power Automate parses US-format EndTime correctly in this tenant. No normalisation fix needed. |
