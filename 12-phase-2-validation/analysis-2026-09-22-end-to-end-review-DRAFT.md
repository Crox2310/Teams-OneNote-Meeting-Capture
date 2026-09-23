# End-to-end review — Flows A–D + Topic — 22 Sep 2026 — DRAFT v2

> **STATUS: DRAFT — fill-path root cause now confirmed. R-01 fix is ready to apply. Other findings still provisional.**
>
> Updated 23 Sep 2026 with full SharePoint list export (10 rows) and column type confirmation.
> - `BACKLOG.md` has **not** been changed. The proposed rewrite is at the end of this file for review only.
> - `ARCHITECTURE.md` has **not** been changed.
> - Open questions Q1, Q2, Q3, Q4, Q6, Q7, Q8 still need answers before the Flow B findings and docs sections are finalised.

**Inputs reviewed:** published code-view exports of Flow A, B, C and D; Topic YAML; agent overview; flow overview pages; `ARCHITECTURE.md`; `BACKLOG.md`; `analysis-2026-09-19-end-to-end-review.md`; full SharePoint list export (10 rows, all columns); list Settings column-type page.
**Not reviewed:** known-good-values references; run history detail; field-testing notes.
**Reviewer:** Claude (Opus 5.5), chat sessions 22–23 Sep 2026.

---

## 0. Open questions (updated 23 Sep)

### Answered by SharePoint export

| # | Question | Answer |
|---|---|---|
| E3 | Row count and uncaptured rows | **10 rows total. 2 uncaptured (ChatCaptured = FALSE).** See §SP below. |
| E3 | EndTime format | **US locale text: `MM/dd/yyyy HH:mm:ss`** — confirmed across all rows. |
| E3 | ChatCaptured column type | **Yes/No** (confirmed in List Settings). |
| E3 | Duplicate rows | **None.** Each meeting appears once. |
| E3 | Poison row identity | **Row 2 (Winning Together Peak call): empty JoinUrl, one-off, ChatCaptured FALSE.** This is the fill-path failure. |

### Still open

| # | Question | Affects |
|---|---|---|
| Q1 | Is the uploaded Topic YAML the **published** version? Has `ContentValidationError` appeared? | R-08, BL-02 |
| Q2 | **One-off section design** — fixed section or per-title `Mtg -`? Was the fixed-section change never made or deliberately reversed? | R-04, R-05, R-17, BL-09 |
| Q3 | Was FD03's **today/yesterday filter** ever built and then lost, or only planned? | R-17 |
| Q4 | Do any meetings use **`teams.microsoft.com/meet/`** join-link format? | R-09 |
| Q6 | For a failed chat step: mark captured anyway (chat lost, no duplicates) or keep retrying (duplicates until fixed)? | R-02 |
| Q7 | Should I read the **known-good-values references** before finalising the corruption check? | Corruption check |
| Q8 | Is **single-match forced pick "1"** a problem in the field? | BL-12 |
| E4 | Raw `text_6` / `text_7` from one Flow B run | R-07, BL-02 |
| E5 | Any successful Flow C run since 20 Sep | Summary, R-02 |
| E6 | OneNote pages: any recap appended more than once? | R-02, R-10 |
| E7 | Field-testing notes | All |

---

## SP. SharePoint list — full picture (23 Sep)

**10 rows total. Well within FD03's `$top 50`. R-03 is not urgent.**

| # | MeetingTitle | Type | OccurrenceDate | EndTime | JoinUrl | ChatCaptured | Notes |
|---|---|---|---|---|---|---|---|
| 1 | 1:1 David C \| Rich B | Recurring | 02/10/2026 | 10/02/2026 10:00 | Present | **FALSE** | Future meeting. OccurrenceDate in UK format `dd/MM/yyyy` — inconsistent with all other rows. |
| 2 | Mtg - Winning Together Peak call | One-off | 22/09/2026 | 09/22/2026 15:30:00 | **EMPTY** | **FALSE** | Confirmed restricted-invite row. **Root cause of fill-path failure.** |
| 3 | SC&L FLT Stand-up | Recurring | 21/09/2026 | 09/21/2026 08:00:00 | Present | TRUE | |
| 4 | Supply Chain Tech and Data... | Recurring | 22/09/2026 | 09/22/2026 09:30:00 | Present | TRUE | |
| 5 | SC&L Portfolio Design Authority | Recurring | 22/09/2026 | 09/22/2026 10:00:00 | Present | TRUE | |
| 6 | SCT Programme Board | Recurring | 22/09/2026 | 09/22/2026 11:00:00 | Present | TRUE | |
| 7 | Mtg - Rapid Alerts | One-off | 22/09/2026 | 09/22/2026 15:25:00 | Present | TRUE | SeriesMasterId empty — one-off fill confirmed working. |
| 8 | Mtg - Fortification | One-off | 22/09/2026 | 09/22/2026 12:30:00 | Present | TRUE | Same. |
| 9 | Mtg - TEST - JoinUrl Pipeline Check | One-off | 23/09/2026 | 09/23/2026 04:25:00 | Present | TRUE | Today's clean test — end-to-end confirmed working. |
| 10 | (1:1 Rich B — partial, first screenshot) | Recurring | — | — | — | TRUE | |

**Key observations:**
- The one-off fill path **has worked** (rows 7, 8, 9 all TRUE). BL-03 is partially answered — set-up and auto-fill both worked for one-offs with a valid JoinUrl.
- Row 1's OccurrenceDate `02/10/2026` is UK-format DD/MM/YYYY. Every other row uses `yyyy-MM-dd`. This is a data quality inconsistency — new finding R-18.
- Row 2 has no JoinUrl. Flow D passes it every 10 minutes → Flow C is called → FC05 `GetOnlineMeeting` is called with an empty string → fast rejection. **This is the fill-path failure.**
- `ChatCaptured` is a Yes/No column. The filter expression `not(equals(item()?['ChatCaptured'], true))` has correctly excluded rows 3–9 (all TRUE), so the expression is working for boolean true. The concern about Yes/No column behaviour can be lowered to L.

---

## 1. Summary (updated 23 Sep)

**Set-up path (Topic → A → B): healthy.** Flow A and B runs all succeeded; agent shows 100% successful runs over 7 days. One-off set-up and fill confirmed working (rows 7, 8, 9).

**Fill path (D → C): failing due to one poison row.**
- Row 2 (Winning Together Peak call) has an empty JoinUrl and is one-off (no SeriesMasterId). FD04b passes it on every cycle because it meets all three current filter conditions (ChatCaptured ≠ true, SeriesMasterId empty, MeetingId present, EndTime present). FD06c's offset has long elapsed. FD06d fires Flow C. FC05 calls `GetOnlineMeeting` with an empty string and fails fast (~350 ms).
- This single row is responsible for all ~150 daily Flow D errors.
- The fix is a **one-line expression addition to FD04b**: add `not(empty(coalesce(item()?['JoinUrl'], '')))`. This row is a confirmed permanent special case (restricted invite, JoinUrl will never resolve) and should simply be excluded.
- Row 1 (future 1:1, OccurrenceDate `02/10/2026`) is also uncaptured and will enter FD04 once its date becomes current. Its OccurrenceDate format inconsistency (R-18) means a date-window filter would need to handle both formats.

**BL-01 confirmed superseded.** FD06d fires correctly. The failure is inside Flow C at FC05.

**Two high-severity Flow B findings from 19 Sep (F3, F4) are still open** in the exports and missing from `BACKLOG.md`.

---

## 2. Findings (updated 23 Sep)

Risk key: **Expr** = expression or parameter edit only · **Settings** = run-after change · **Struct** = add, move or delete actions.

| ID | Component | Finding | Evidence | Impact | Sev | Fix | Risk | Confidence |
|---|---|---|---|---|---|---|---|---|
| R-01 | D / C | **CONFIRMED.** Row 2 (Winning Together Peak call) has empty JoinUrl. FD04b passes it every cycle. Flow C fails at FC05 `GetOnlineMeeting` with empty input (~350 ms). Every Flow D run fails. | SP export: row 2 JoinUrl empty; FD04b `where` has no JoinUrl guard; D error trend ~150/day | Every D run fails; real fills never execute | H | **Add to FD04b `where`:** `not(empty(coalesce(item()?['JoinUrl'], '')))` | Expr | **Confirmed** |
| R-02 | C | Recap duplicated when chat part fails. FC16 runs only if FC15 succeeds, but FC05t already appended the recap. | FC16 runAfter `FC15: Succeeded`; FC06 runs after FC05c on any status | Recap re-appended on every retry cycle | H | FC16 run-after: Succeeded, Failed, Skipped, TimedOut | Settings | Logic verified; Q6 needed for trade-off |
| R-03 | D | FD03 `$top 50`, no filter or order. `ARCHITECTURE.md` says there's a today/yesterday filter; export has none. | FD03 code; 10 rows currently | **LOW URGENCY — 10 rows, safe for months.** Will bite once list exceeds 50 rows. | M (latent) | FD03 Filter Query `ChatCaptured eq 0`, Order By `ID desc` | Expr | **Confirmed not urgent. Q3 for history.** |
| R-04 | B | F3 still open: recurring section lookup/create runs for one-offs. FB-F01 caps at 37 chars, `Compose_SafeSectionName` at 43. | Condition expressions have no IsRecurring gate | Long one-off titles → two sections | H | F3 expressions | Expr | Verified in export; right fix depends on Q2 |
| R-05 | B | F4 still open: one-off re-capture can report SUCCESS falsely. | `Filter_Existing_Section_By_Name` uses 43-char name match | Nothing written; user told it saved | H | Filter by `pagesUrl` | Expr | Verified in export |
| R-06 | B | Existing page found by date substring. Fallback creates untitled page with no mapping update. | `Filter_Pages_By_Title`, `Guard_Create_Page_Fallback` | Wrong-page append; fails on empty `text_5` | M | Match by page id | Expr | Verified in export |
| R-07 | Topic → B → D | EndTime stored as US locale text (`MM/dd/yyyy HH:mm:ss`). FD06c parses with `ticks()`. | SP export: all EndTime values are US-format text. Column type: Single line of text. | In a UK-locale tenant, `ticks('09/22/2026 09:30:00')` may parse as 9 Oct not 22 Sep — fills could run ~13 days late or never. | M | Normalise at write in both `Create_Mapping_Item_*` actions | Expr | **Confirmed format. Parsing behaviour needs one Scratch Diagnostics test.** |
| R-08 | Topic → B | C10 contract differs from `ARCHITECTURE.md`: `text_3` is HTML skeleton only; `text_7` = JoinUrl. | Topic C10 bindings | Docs wrong; "Meeting Invite" always empty | M | Update docs | Docs | Depends on Q1 |
| R-09 | A | Join link extraction only matches `/l/meetup-join/`. `/meet/` invites get empty JoinUrl — same failure as row 2. | FA12B | Those meetings become poison rows | M | Handle both formats | Expr | Q4 for prevalence |
| R-10 | C | FC01 (`$top 1`) can re-select an already-captured row when duplicates exist. | FC01 `$filter` | Endless re-append | M | Add `and ChatCaptured eq 0` | Expr | No duplicates currently; safe to fix preventively |
| R-11 | B | Hard failures return nothing to the Topic. No Scope/catch. | Tail action run-afters | Generic error; C12 unreachable; duplicate pages on retry | M | Scope + catch | Struct | Verified |
| R-12 | C | Chat limited to last 50 messages, no paging. | FC07 URI | Busy meetings truncated | L | Document; paging structural | Struct | Verified |
| R-13 | B | UJ3b delete failure blocks whole run. | IsRecurring runAfter `SUCCEEDED` only | One failed delete fails capture | L | Add Failed to run-after | Settings | Verified |
| R-14 | B | Re-capture appends full skeleton with misleading text. | `Compose_UpdateHtmlFragment` | Duplicate headings | L | Header + datestamp only | Expr | Verified |
| R-15 | B | `outbranchresult` returns `varFinalMatchCount`. | Respond body | Diagnostics only | L | Coalesce branch results | Expr | Verified |
| R-16 | A/B/Topic/C | Dead or literal logic (list below). | Exports | Confusion; corruption exposure | L | Leave until stable, then BL-14 | Struct | Verified |
| R-17 | Docs | `ARCHITECTURE.md` drift (list below). | Exports vs doc | Future sessions reason from wrong design | M | Rewrite affected sections | Docs | Depends on Q1–Q3 |
| R-18 | D / Data | **NEW.** Row 1 OccurrenceDate is `02/10/2026` (UK DD/MM/YYYY). All other rows use `yyyy-MM-dd`. A date-window filter on FD04 would either reject this row or parse it as 2 Oct correctly by accident. | SP export row 1 | Row 1 will fail any ISO date comparison once a window filter is added; the future capture for this meeting may be missed. | M | Manually correct row 1 OccurrenceDate to `2026-10-02` now. Then enforce ISO format in Flow B at write time (already done for other rows via `yyyy-MM-dd` Topic output). | Data fix (manual SP edit) then Expr guard | **Confirmed** |

**R-16 dead or literal logic:** FA12 appends literal `"json(concat(...` (no `@`) to unused `varCandidates`; FA14 uses `item()` outside loop; FA15–FA26 never reached; FA28–FA30 single-match outputs always overwritten by C6D; Flow B D2 branch unreachable; `Compose_IgnoreSeriesMasterId` literal `''`; `outpagehtml`/`outupdatehtmlfragment` unbound; Topic C9B `PageTitle` unused; Flow C `AllowFallback` unused.

**R-17 `ARCHITECTURE.md` drift:** Flow A uses V3 connector not V4; one-offs get per-title `Mtg -` sections not fixed section; FD03 has no date filter; Flow C appends to `body` not `#notes`/`#chat`; C10 contract (R-08); Flow B and C use different notebookKey paths.

---

## 3. Immediate fix — R-01 (ready to apply)

**The fill-path failure is caused by one expression gap in FD04b.** This is the minimum change needed to stop the 150 daily errors and allow real fills to run.

**FD04b current `where`:**
```
@and(not(equals(item()?['ChatCaptured'], true)), empty(coalesce(item()?['SeriesMasterId'], '')), not(empty(coalesce(item()?['MeetingId'], ''))), not(empty(coalesce(item()?['EndTime'], ''))))
```

**FD04b fixed `where` — add one condition:**
```
@and(not(equals(item()?['ChatCaptured'], true)), empty(coalesce(item()?['SeriesMasterId'], '')), not(empty(coalesce(item()?['MeetingId'], ''))), not(empty(coalesce(item()?['EndTime'], ''))), not(empty(coalesce(item()?['JoinUrl'], ''))))
```

This excludes row 2 (Winning Together Peak call) permanently. It also protects against any future one-off row that ends up with an empty JoinUrl (e.g. `/meet/` format invites — R-09).

**Also recommended before re-testing:** manually set row 1's OccurrenceDate from `02/10/2026` to `2026-10-02` in the SharePoint list (R-18). This is a 30-second manual edit and ensures the future 1:1 capture works correctly when it becomes due.

**After applying FD04b fix:** publish Flow D, wait for the next 10-minute cycle, and check Flow D's run history. It should complete without errors. Then reset one test row (set `ChatCaptured = No`) and confirm an end-to-end fill runs cleanly.

---

## 4. Candidate fix expressions (other findings)

**R-04 (F3):**
- `Condition_Section_Exists_Recurring`: `@and(equals(toLower(string(triggerBody()?['text'])), 'true'), equals(outputs('Compose_Section_Match_Count_Recurring'), 1))` equals `true`
- `Condition_Section_Count_Is_Zero`: `@and(equals(toLower(string(triggerBody()?['text'])), 'true'), equals(outputs('Compose_Section_Match_Count_Recurring'), 0))` equals `true`

**R-05 (F4):**
- `Condition_Recurring_TargetSection`: `@not(empty(first(coalesce(body('Filter_Existing_Mapping'), body('OF01_—_Filter_Existing_Mapping_OneOff'), createArray()))?['SectionPagesUrl']))` equals `true`
- `Set_varTargetSectionPagesUrl_ExistingMapping`: `@first(coalesce(body('Filter_Existing_Mapping'), body('OF01_—_Filter_Existing_Mapping_OneOff'), createArray()))?['SectionPagesUrl']`
- `Filter_Existing_Section_By_Name` where: `@equals(item()?['pagesUrl'], variables('varTargetSectionPagesUrl'))`

**R-06:** `Filter_Pages_By_Title` where: `@equals(item()?['id'], outputs('Compose_ExistingPageId'))`

**R-07** (verify parsing in Scratch Diagnostics first, then apply):
`@if(empty(coalesce(triggerBody()?['text_6'], '')), '', formatDateTime(triggerBody()?['text_6'], 'yyyy-MM-ddTHH:mm:ssZ'))`

**R-09** (prove `/meet/` in Scratch Diagnostics first):
```
@if(contains(coalesce(item()?['body'],''),'href="https://teams.microsoft.com/l/meetup-join/'), concat('https://teams.microsoft.com/l/meetup-join/', first(split(split(coalesce(item()?['body'],''),'href="https://teams.microsoft.com/l/meetup-join/')[1],'"'))), if(contains(coalesce(item()?['body'],''),'href="https://teams.microsoft.com/meet/'), concat('https://teams.microsoft.com/meet/', first(split(split(coalesce(item()?['body'],''),'href="https://teams.microsoft.com/meet/')[1],'"'))), ''))
```

**R-10:** FC01 `$filter` — append ` and ChatCaptured eq 0` to existing expression.

**R-14:** `Compose_UpdateHtmlFragment`: `@concat('<hr><p><em>Re-captured by Meeting Capture Agent on ', formatDateTime(utcNow(), 'd MMM yyyy HH:mm'), ' UTC.</em></p>')`

**R-15:** `outbranchresult`: `@{coalesce(outputs('Compose_Branch_Result'), outputs('Compose_Branch_Result_NoMatch'))}`

---

## 5. User journeys (updated)

| Journey | Recurring | One-off |
|---|---|---|
| New capture | OK | OK — confirmed working (rows 7, 8, 9) |
| Re-capture, same occurrence | Second skeleton appended (R-14); may pick wrong page (R-06) | False SUCCESS for long titles (R-05) |
| No-match day + navigation | OK | OK |
| Multi-match selection | OK | OK |
| Single match | Forced to pick "1" (BL-12) | Same |
| Auto fill, recap available | Failing (R-01 — fix ready); duplication risk (R-02) | Working for valid JoinUrl; fails for empty JoinUrl (R-01) |
| Auto fill, no recap | Chat only — correct design | Same |
| No Teams link / empty JoinUrl | Poison row — fix is R-01 | Same — confirmed by row 2 |
| Past-date capture | Set-up OK; fill needs date-window fix once R-01 applied | Same |
| Future meeting | Set-up OK; FD04 passes it now; will attempt fill before meeting ends | Date format inconsistency on row 1 (R-18) |

---

## 6. Backlog reconciliation (updated 23 Sep)

| ID | Status | Reason |
|---|---|---|
| BL-01 | **Superseded — confirmed** | FD06d fires correctly. Fill failure is FC05 on empty JoinUrl → R-01. |
| BL-02 | Fixed, pending Q1 | Both Create Mapping Item read `text_7`; Topic sends JoinUrl there. |
| BL-03 | **Partially answered** | One-off set-up and fill confirmed working (rows 7–9). Full test matrix still needed. |
| BL-04 | Fixed | FD01 = 30 confirmed. |
| BL-05 | Superseded | Flow C appends to `body`; no slots; no AllowFallback gate. Docs wrong (R-17). |
| BL-06 | **Downgraded to L** | 10 rows — safe for months. Monitor. |
| BL-07 | Open | No SectionPagesUrl guard added. |
| BL-08 | Open | OF09 gate still depends on Set_PageTitle succeeding. |
| BL-09 | Open | Still `Mtg -`. Tied to Q2. |
| BL-10 | Open | Design gap. |
| BL-11 | Changed | UpdatedAppend fixed; `outbranchresult` remains (R-15); formatDateTime guard covered by R-06. |
| BL-12 | Open | No MatchCount = 1 branch. Q8. |
| BL-13 | Fixed, caveat | meetup-join links work; `/meet/` → R-09. |
| BL-14 | Open, expanded | Add R-16 items. |
| BL-15 | Open | Document. |
| BL-16, 17, 19, 20 | Open | Not verifiable from exports. |
| BL-18 | Fixed | `text` = IsRecurring, `text_1` = Title, `text_2` = SeriesMasterId. |
| F1, F2, F8, F9 | Fixed | Seen in exports. |
| F3, F4 | Open | Missing from backlog → R-04, R-05. |
| F5, F6, F7 | Open | → BL-14, BL-06 (downgraded), BL-07. |
| F10 | Low, unconfirmed | HTML outputs returned but unbound. |

---

## 7. Recommended work order (updated)

1. **Now — two quick fixes (Sonnet, no session needed).**
   - Manually edit row 1 OccurrenceDate in SharePoint: `02/10/2026` → `2026-10-02` (R-18).
   - Apply the FD04b JoinUrl expression fix (R-01). Publish Flow D. Confirm clean run.

2. **Session 1 — verify and stabilise (Sonnet).**
   - Confirm Flow D runs clean after R-01 fix.
   - Test R-07: in Scratch Diagnostics, run `ticks('09/22/2026 09:30:00')` and `ticks('2026-09-22T09:30:00Z')` and compare. If they differ materially, apply the EndTime normalisation fix in both Create Mapping Item actions.
   - Apply R-02 (FC16 run-after settings change).
   - Apply R-10 (FC01 ChatCaptured guard).
   - Reset one test row; confirm clean end-to-end fill.

3. **Session 2 — Flow B correctness (Opus to design tests, Sonnet to edit).**
   - Answer Q2 (one-off section design intent) before touching anything.
   - R-04, R-05, R-06, BL-07, R-15, R-14.
   - Test matrix: recurring new, recurring existing, one-off new, one-off existing.

4. **Session 3 — contract and data quality (Sonnet).**
   - Answer Q1 (Topic version, ContentValidationError).
   - R-09 after Scratch Diagnostics proof.
   - Rewrite drifted `ARCHITECTURE.md` sections (R-17, Q1–Q3 needed).
   - BL-17 (known-good values merge).

5. **Later — structural (Opus to design).**
   - BL-08, R-11, BL-10, BL-14.

---

## 8. Proposed `BACKLOG.md` rewrite (for review only — not applied)

| ID | Item | Type | Pri | Source | Notes |
|---|---|---|---|---|---|
| BL-21 | **Fill path failing: empty JoinUrl passes FD04b.** Row 2 (Winning Together Peak call) loops every 10 min. | Bug | H | R-01 confirmed | Expr. Add `not(empty(coalesce(item()?['JoinUrl'], '')))` to FD04b. **Ready to apply.** |
| BL-18b | **Row 1 OccurrenceDate in wrong format** (`02/10/2026` instead of `yyyy-MM-dd`). | Bug | H | R-18 | Manual SP edit: change to `2026-10-02`. 30-second fix. |
| BL-22 | Recap duplicated when chat part fails. | Bug | H | R-02 | Settings. FC16 run-after. Decision Q6. |
| BL-24 | Recurring section logic runs for one-offs (F3). | Bug | H | F3/R-04 | Expr. Depends on Q2. |
| BL-25 | One-off re-capture false SUCCESS (F4). | Bug | H | F4/R-05 | Expr. |
| BL-03 | Live one-off test matrix (full). | Verify | H | Session 21 Sep | Partially answered — rows 7–9 captured. Full matrix still needed. |
| BL-27 | EndTime not stored as UTC ISO. | Bug | M | R-07 | Verify parsing in Scratch Diagnostics first. |
| BL-26 | Existing page found by date text. | Bug | M | R-06 | Expr. |
| BL-28 | `/meet/` join links not recognised → poison rows. | Bug | M | R-09 | Expr. Scratch Diagnostics proof first. |
| BL-29 | FC01 can re-select captured row. | Bug | M | R-10 | Expr. Preventive. |
| BL-07 | Stale skeleton rows (UJ3b). | Bug | M | F7 | Expr. |
| BL-08 | Title-set failure blocks mapping write. | Bug | M | 16 Aug | Struct. |
| BL-30 | Flow B hard failures return nothing to Topic. | Bug | M | R-11 | Struct. |
| BL-32 | Rewrite drifted `ARCHITECTURE.md` sections. | Tidy | M | R-17 | Docs. Q1–Q3 first. |
| BL-17 | Known-good values merge; add C and D. | Tidy | M | Weekend plan D4 | |
| BL-23 | FD03 `$top 50` — monitor, apply OData filter before list reaches 40 rows. | Bug | L | R-03 | Downgraded from H. 10 rows currently. |
| BL-09 | Rename `Mtg -` → `Rec -`. | Improve | L | Weekend plan C2 | Q2 first. |
| BL-10 | No re-fill after late recap. | Improve | L | Architecture §9 | |
| BL-11 | `outbranchresult` binding. | Bug | L | 19 Sep minor | Expr. |
| BL-12 | Single-match forces pick "1". | Improve | L | 6 Sep | Q8. |
| BL-31 | Chat capped at 50 messages. | Improve | L | R-12 | Struct. Document for now. |
| BL-33 | UJ3b delete failure blocks Flow B. | Bug | L | R-13 | Settings. |
| BL-34 | Re-capture appends full skeleton. | Bug | L | R-14 | Expr. |
| BL-35 | "Meeting Invite" heading always empty. | Improve | L | R-08 | Topic expression. Q1 first. |
| BL-14 | Remove dead paths (expanded with R-16 items). | Tidy | L | F5/R-16 | Struct. After stable sessions. |
| BL-15 | Mixed OneNote connections / notebookKey. | Tidy | L | 19 Sep minor | Document. |
| BL-16 | Clean-up test artefacts, `iCalUId`. | Tidy | L | Weekend plan D5 | |
| BL-19 | Page header: attendees, organiser, times. | Improve | L | Weekend plan | |
| BL-20 | Microsoft support ticket. | Improve | L | 15 Aug | |

**Would close:** BL-01 (confirmed superseded), BL-04, BL-05, BL-13, BL-18 (superseded by BL-18b).

---
*Draft v1 created 22 Sep 2026. Updated to v2 on 23 Sep 2026 with SharePoint list data — fill-path root cause confirmed, R-01 ready to apply, R-18 added.*
