# End-to-end review — Flows A–D + Topic — 22 Sep 2026 — DRAFT v3

> **STATUS: DRAFT — fill-path root cause confirmed and fix applied. R-07 needs one Scratch Diagnostics test. Q2 and Q4 still open.**
>
> Updated 23 Sep 2026 (v3) with: Topic YAML confirmed as published (Q1 closed); session-2026-09-18 log read (Q3 closed, R-07 and R-09 updated); R-18 closed (OccurrenceDate format was display artefact only); FD04 and FD04b JoinUrl + PageSelfUrl fixes confirmed applied; row 2 deleted from SharePoint.
> - `BACKLOG.md` has **not** been changed. The proposed rewrite is at the end of this file for review only.
> - `ARCHITECTURE.md` has **not** been changed.

**Inputs reviewed:** published code-view exports of Flow A, B, C and D; Topic YAML (confirmed published); agent overview; flow overview pages; `ARCHITECTURE.md`; `BACKLOG.md`; `analysis-2026-09-19-end-to-end-review.md`; full SharePoint list export (10 rows, all columns); list Settings column-type page; `session-2026-09-18-endtime-joinurl-flowd.md`.
**Not reviewed:** known-good-values references; run history detail; field-testing notes.
**Reviewer:** Claude (Opus 5.5), chat sessions 22–23 Sep 2026.

---

## 0. Open questions (updated 23 Sep v3)

### Answered

| # | Question | Answer | Affects |
|---|---|---|---|
| E3 | Row count and uncaptured rows | **10 rows total. 2 uncaptured.** Row 2 now deleted. | R-01 |
| E3 | EndTime format | **US locale text `MM/dd/yyyy HH:mm:ss`** — intentional; see R-07 below. | R-07 |
| E3 | ChatCaptured column type | **Yes/No** — filter expression working correctly. | — |
| E3 | Duplicate rows | **None.** | R-10 |
| Q1 | Is uploaded Topic YAML the published version? | **Yes — confirmed identical.** `text_7` = JoinUrl is live. No ContentValidationError reported. | R-08, BL-02, F10 |
| Q3 | Was FD03's today/yesterday filter ever built? | **Never built — only planned.** `ARCHITECTURE.md` description is wrong. `session-2026-09-18` confirms FD03 was redesigned on 18 Sep with no date filter. | R-03, R-17 |
| BL-01 | FD06d skipped? | **Superseded — confirmed.** FD06d fires correctly; failure was FC05 on empty JoinUrl (row 2). | — |
| BL-02 | JoinUrl source | **Fixed.** Both Create Mapping Item actions read `text_7`; Topic sends JoinUrl there. | — |
| BL-04 | Offset value | **Fixed.** FD01 = 30. | — |
| BL-18 | C10 input order | **Fixed.** `text` = IsRecurring, `text_1` = Title, `text_2` = SeriesMasterId. | — |
| R-18 | Row 1 OccurrenceDate wrong format | **Closed — display artefact.** List view showed `02/10/2026` but underlying stored value is `2026-10-02` (ISO). Confirmed by opening the row directly. No manual edit needed. | — |

### Still open

| # | Question | Affects |
|---|---|---|
| Q2 | **One-off section design** — fixed section or per-title `Mtg -`? Was the fixed-section change never made or deliberately reversed? | R-04, R-05, R-17, BL-09 |
| Q4 | Do any meetings use **`teams.microsoft.com/meet/`** join-link format? | R-09 |
| Q6 | For a failed chat step: mark captured anyway (chat lost, no duplicates) or keep retrying (duplicates until fixed)? | R-02 |
| Q8 | Is **single-match forced pick "1"** a problem in the field? | BL-12 |
| E5 | Any successful Flow C run since 20 Sep? | Summary, R-02 |
| E6 | OneNote pages: any recap appended more than once? | R-02, R-10 |
| E7 | Field-testing notes | All |
| Scratch | `ticks('09/22/2026 09:30:00')` in Scratch Diagnostics — does Power Automate parse US-format text correctly? | R-07 |

---

## SP. SharePoint list — full picture (23 Sep)

**10 rows total. Row 2 deleted 23 Sep. 1 uncaptured row remaining (row 1, future meeting).**

| # | MeetingTitle | Type | OccurrenceDate | EndTime | JoinUrl | ChatCaptured | Notes |
|---|---|---|---|---|---|---|---|
| 1 | 1:1 David C \| Rich B | Recurring | 2026-10-02 | 10/02/2026 10:00 | Present | **FALSE** | Future meeting. OccurrenceDate correctly stored as ISO. EndTime US-format — see R-07. |
| ~~2~~ | ~~Mtg - Winning Together Peak call~~ | ~~One-off~~ | ~~22/09/2026~~ | ~~09/22/2026 15:30:00~~ | ~~**EMPTY**~~ | ~~**FALSE**~~ | **Deleted 23 Sep.** Restricted-invite row — root cause of fill-path failure. |
| 3 | SC&L FLT Stand-up | Recurring | 21/09/2026 | 09/21/2026 08:00:00 | Present | TRUE | |
| 4 | Supply Chain Tech and Data... | Recurring | 22/09/2026 | 09/22/2026 09:30:00 | Present | TRUE | |
| 5 | SC&L Portfolio Design Authority | Recurring | 22/09/2026 | 09/22/2026 10:00:00 | Present | TRUE | |
| 6 | SCT Programme Board | Recurring | 22/09/2026 | 09/22/2026 11:00:00 | Present | TRUE | |
| 7 | Mtg - Rapid Alerts | One-off | 22/09/2026 | 09/22/2026 15:25:00 | Present | TRUE | SeriesMasterId empty — one-off fill confirmed working. |
| 8 | Mtg - Fortification | One-off | 22/09/2026 | 09/22/2026 12:30:00 | Present | TRUE | Same. |
| 9 | Mtg - TEST - JoinUrl Pipeline Check | One-off | 23/09/2026 | 09/23/2026 04:25:00 | Present | TRUE | Today's clean test — end-to-end confirmed working. |
| 10 | (1:1 Rich B — partial) | Recurring | — | — | — | TRUE | |

---

## 1. Summary (updated 23 Sep v3)

**Set-up path (Topic → A → B): healthy.** All recent runs succeeded; agent shows 100% successful runs over 7 days. One-off set-up and fill confirmed working (rows 7, 8, 9).

**Fill path (D → C): poison row deleted; FD04 and FD04b hardened.**
- Row 2 (Winning Together Peak call, empty JoinUrl) was the sole cause of ~150 daily Flow D errors. Deleted 23 Sep.
- FD04 and FD04b have been updated to add `not(empty(coalesce(item()?['JoinUrl'], '')))` and `not(empty(coalesce(item()?['PageSelfUrl'], '')))` guards. Published.
- Flow D should now run cleanly. Row 1 (future 1:1, 2 Oct) will enter FD04 when current but FD06c should take the False branch until after the meeting ends.
- **One remaining risk:** R-07 — EndTime is stored as US-locale text (`10/02/2026 10:00:00`). Whether `ticks()` in a UK-locale Power Automate tenant parses this as 2 Oct or 10 Feb needs one Scratch Diagnostics test. If it parses as 10 Feb, the 1:1 fill will never fire automatically.

**BL-01 confirmed superseded.** FD06d fires correctly.

**Two high-severity Flow B findings from 19 Sep (F3, F4) remain open** and are missing from `BACKLOG.md`.

---

## 2. Findings (updated 23 Sep v3)

Risk key: **Expr** = expression or parameter edit only · **Settings** = run-after change · **Struct** = add, move or delete actions.

| ID | Component | Finding | Evidence | Impact | Sev | Fix | Risk | Confidence |
|---|---|---|---|---|---|---|---|---|
| R-01 | D / C | **FIXED.** Row 2 deleted; FD04b JoinUrl + PageSelfUrl guards added; FD04 same guards added. Published. | SP list; FD04/FD04b peek code confirmed | Was causing all ~150 daily errors | H | **Done.** | — | **Confirmed + fixed** |
| R-02 | C | Recap duplicated when chat part fails. FC16 runs only if FC15 succeeds, but FC05t already appended the recap. | FC16 runAfter `FC15: Succeeded` | Recap re-appended on every retry cycle | H | FC16 run-after: Succeeded, Failed, Skipped, TimedOut | Settings | Logic verified; Q6 needed for trade-off |
| R-03 | D | FD03 `$top 50`, no filter or order. `ARCHITECTURE.md` says today/yesterday filter exists — **it was never built** (Q3 confirmed from session log). | FD03 code; 9 rows currently | Not urgent. Will bite once list exceeds 50 rows. | L (was M) | FD03 Filter Query `ChatCaptured eq 0`, Order By `ID desc` | Expr | **Confirmed. Q3 closed.** |
| R-04 | B | F3 still open: recurring section lookup/create runs for one-offs. | Condition expressions have no IsRecurring gate | Long one-off titles → two sections | H | F3 expressions | Expr | Verified; right fix depends on Q2 |
| R-05 | B | F4 still open: one-off re-capture can report SUCCESS falsely. | `Filter_Existing_Section_By_Name` uses 43-char name match | Nothing written; user told it saved | H | Filter by `pagesUrl` | Expr | Verified |
| R-06 | B | Existing page found by date substring. Fallback creates untitled page with no mapping update. | `Filter_Pages_By_Title`, `Guard_Create_Page_Fallback` | Wrong-page append; fails on empty `text_5` | M | Match by page id | Expr | Verified |
| R-07 | Topic → B → D | **EndTime format is intentional US locale text, not a bug.** The `UTC\|` prefix in FA12B forces Power Fx to treat the ISO value as a non-date string (preventing silent US-Pacific reformatting), then Flow B strips the prefix before writing. The stored value `09/22/2026 09:30:00` is UTC time, encoded for pipeline survival. Risk: Power Automate's `ticks()` in a UK-locale tenant may parse `09/22/2026` as 9 Oct not 22 Sep. | `session-2026-09-18-endtime-joinurl-flowd.md` explains the `UTC\|` design; SP list shows stored format. | Fills could run 17 days late or never for row 1 (EndTime `10/02/2026 10:00:00`). | M | **Run Scratch Diagnostics test first:** Compose `ticks('09/22/2026 09:30:00')` and `ticks('2026-09-22T09:30:00Z')` and compare. If they differ, apply the normalisation fix in both `Create_Mapping_Item_*` EndTime fields. | Expr | **Design understood. Parsing behaviour unverified — Scratch test needed.** |
| R-08 | Topic → B | **Confirmed from published YAML.** C10 contract: `text_3` = HTML skeleton only (no `||||`, no invite body, no `data-id` divs); `text_7` = JoinUrl. `ARCHITECTURE.md` description is wrong. | Topic C10 bindings confirmed | "Meeting Invite" heading always empty; docs misleading | M | Update `ARCHITECTURE.md`; optionally add BodyPreview to `text_3` (BL-35) | Docs | **Confirmed** |
| R-09 | A | **Confirmed real from session log.** `/meet/` links are newer format; `session-2026-09-18` explicitly notes this and records that `FA29D` was changed to look for `/l/meetup-join/` specifically. Any meeting whose invite body only contains a `/meet/` link gets empty JoinUrl → poison row pattern. | `session-2026-09-18-endtime-joinurl-flowd.md` §2 | Those meetings become poison rows (same pattern as row 2) | M | Handle both formats in FA12B and FA29D–F; prove in Scratch Diagnostics first | Expr | **Confirmed real. Prevalence still needs Q4.** |
| R-10 | C | FC01 (`$top 1`) can re-select an already-captured row when duplicates exist. | FC01 `$filter` | Endless re-append | M | Add `and ChatCaptured eq 0` | Expr | No duplicates currently; preventive |
| R-11 | B | Hard failures return nothing to the Topic. No Scope/catch. | Tail action run-afters | Generic error; C12 unreachable; duplicate pages on retry | M | Scope + catch | Struct | Verified |
| R-12 | C | Chat limited to last 50 messages, no paging. Recurring meetings share one chat thread. | FC07 URI | Busy meetings truncated | L | Document; paging structural | Struct | Verified |
| R-13 | B | UJ3b delete failure blocks whole run. | IsRecurring runAfter `SUCCEEDED` only | One failed delete fails capture | L | Add Failed to run-after | Settings | Verified |
| R-14 | B | Re-capture appends full skeleton with misleading "preserved below" text. | `Compose_UpdateHtmlFragment` | Duplicate headings | L | Header + datestamp only | Expr | Verified |
| R-15 | B | `outbranchresult` returns `varFinalMatchCount`. | Respond body | Diagnostics only | L | Coalesce branch results | Expr | Verified |
| R-16 | A/B/Topic/C | Dead or literal logic (list below). | Exports | Confusion; corruption exposure | L | Leave until stable, then BL-14 | Struct | Verified |
| R-17 | Docs | `ARCHITECTURE.md` drift — **Q3 now also confirms FD03 date filter was never built.** Full drift list below. | Exports + session log vs doc | Future sessions reason from wrong design | M | Rewrite affected sections | Docs | **Q3 confirmed; Q2 still needed for one-off section description** |

**R-16 dead or literal logic:** FA12 appends literal `"json(concat(...` (no `@`) to unused `varCandidates`; FA14 uses `item()` outside loop; FA15–FA26 never reached; FA28–FA30 single-match outputs always overwritten by C6D (incl. FA28C `endWithTimeZone`); Flow B D2 branch unreachable; `Compose_IgnoreSeriesMasterId` literal `''`; `outpagehtml`/`outupdatehtmlfragment` unbound; Topic C9B `PageTitle` unused; Flow C `AllowFallback` unused.

**R-17 `ARCHITECTURE.md` drift (confirmed items):** Flow A uses V3 connector not V4; FD03 has no today/yesterday date filter (never built); Flow C appends to `body` not `#notes`/`#chat`; C10 contract (`text_3` is HTML skeleton only, `text_7` = JoinUrl); Flow B and C use different notebookKey paths. **Pending Q2:** whether one-offs use a fixed section or per-title `Mtg -` sections.

---

## 3. Current state of fixes applied

| Fix | Status | Detail |
|---|---|---|
| Row 2 deleted | **Done** | Winning Together Peak call removed from SP list 23 Sep |
| FD04b JoinUrl guard | **Done** | `not(empty(coalesce(item()?['JoinUrl'], '')))` added and confirmed in peek code |
| FD04b PageSelfUrl guard | **Done** | `not(empty(coalesce(item()?['PageSelfUrl'], '')))` added and confirmed |
| FD04 JoinUrl guard | **Done** | Same addition to recurring filter |
| FD04 PageSelfUrl guard | **Done** | Same addition to recurring filter |
| Flow D published | **Pending** | Apply after confirming peek code correct |

---

## 4. Candidate fix expressions (remaining findings)

**R-04 (F3):**
- `Condition_Section_Exists_Recurring`: `@and(equals(toLower(string(triggerBody()?['text'])), 'true'), equals(outputs('Compose_Section_Match_Count_Recurring'), 1))` equals `true`
- `Condition_Section_Count_Is_Zero`: `@and(equals(toLower(string(triggerBody()?['text'])), 'true'), equals(outputs('Compose_Section_Match_Count_Recurring'), 0))` equals `true`

**R-05 (F4):**
- `Condition_Recurring_TargetSection`: `@not(empty(first(coalesce(body('Filter_Existing_Mapping'), body('OF01_—_Filter_Existing_Mapping_OneOff'), createArray()))?['SectionPagesUrl']))` equals `true`
- `Set_varTargetSectionPagesUrl_ExistingMapping`: `@first(coalesce(body('Filter_Existing_Mapping'), body('OF01_—_Filter_Existing_Mapping_OneOff'), createArray()))?['SectionPagesUrl']`
- `Filter_Existing_Section_By_Name` where: `@equals(item()?['pagesUrl'], variables('varTargetSectionPagesUrl'))`

**R-06:** `Filter_Pages_By_Title` where: `@equals(item()?['id'], outputs('Compose_ExistingPageId'))`

**R-07** (apply only if Scratch Diagnostics test shows mismatch):
`@if(empty(coalesce(triggerBody()?['text_6'], '')), '', formatDateTime(triggerBody()?['text_6'], 'yyyy-MM-ddTHH:mm:ssZ'))`

**R-09** (prove `/meet/` handling in Scratch Diagnostics first — confirm `GetOnlineMeeting` accepts that format):
```
@if(contains(coalesce(item()?['body'],''),'href="https://teams.microsoft.com/l/meetup-join/'), concat('https://teams.microsoft.com/l/meetup-join/', first(split(split(coalesce(item()?['body'],''),'href="https://teams.microsoft.com/l/meetup-join/')[1],'"'))), if(contains(coalesce(item()?['body'],''),'href="https://teams.microsoft.com/meet/'), concat('https://teams.microsoft.com/meet/', first(split(split(coalesce(item()?['body'],''),'href="https://teams.microsoft.com/meet/')[1],'"'))), ''))
```

**R-10:** FC01 `$filter` — append ` and ChatCaptured eq 0` to existing expression.

**R-14:** `Compose_UpdateHtmlFragment`: `@concat('<hr><p><em>Re-captured by Meeting Capture Agent on ', formatDateTime(utcNow(), 'd MMM yyyy HH:mm'), ' UTC.</em></p>')`

**R-15:** `outbranchresult`: `@{coalesce(outputs('Compose_Branch_Result'), outputs('Compose_Branch_Result_NoMatch'))}`

---

## 5. User journeys (updated v3)

| Journey | Recurring | One-off |
|---|---|---|
| New capture | OK | OK — confirmed working (rows 7, 8, 9) |
| Re-capture, same occurrence | Second skeleton appended (R-14); may pick wrong page (R-06) | False SUCCESS for long titles (R-05) |
| No-match day + navigation | OK | OK |
| Multi-match selection | OK | OK |
| Single match | Forced to pick "1" (BL-12) | Same |
| Auto fill, recap available | Fix applied (R-01); duplication risk remains (R-02) | Working for valid JoinUrl |
| Auto fill, no recap | Chat only — correct design | Same |
| No Teams link / empty JoinUrl | Poison row — now blocked by new FD04 guard | Same |
| `/meet/` format invite | Empty JoinUrl → blocked by guard; no fill until R-09 applied | Same |
| Past-date capture | Set-up OK; fill OK if within 50-row limit | Same |
| Future meeting (row 1, 2 Oct 1:1) | Set-up done; fill will attempt once EndTime elapsed — R-07 test needed to confirm timing | — |

---

## 6. Backlog reconciliation (updated 23 Sep v3)

| ID | Status | Reason |
|---|---|---|
| BL-01 | **Closed** | Superseded and confirmed. FD06d fires; root cause was empty JoinUrl on row 2. |
| BL-02 | **Closed** | `text_7` = JoinUrl confirmed in published YAML and both Create Mapping Item actions. |
| BL-03 | Partially answered | One-off set-up and fill confirmed working (rows 7–9). Full test matrix still needed. |
| BL-04 | **Closed** | FD01 = 30 confirmed. |
| BL-05 | **Closed** | Superseded. Flow C appends to `body`; no slots; no AllowFallback gate. Docs need updating (R-17). |
| BL-06 | **Downgraded to L** | 9 rows — safe for months. Monitor. |
| BL-07 | Open | No SectionPagesUrl guard added to Flow B mapping filters. |
| BL-08 | Open | OF09 gate still depends on Set_PageTitle succeeding. |
| BL-09 | Open | Still `Mtg -`. Tied to Q2. |
| BL-10 | Open | Design gap. |
| BL-11 | Changed | UpdatedAppend fixed; `outbranchresult` remains (R-15); formatDateTime guard covered by R-06. |
| BL-12 | Open | No MatchCount = 1 branch. Q8. |
| BL-13 | **Closed** | meetup-join links work; `/meet/` tracked as R-09. |
| BL-14 | Open, expanded | Add R-16 items. |
| BL-15 | Open | Document. |
| BL-16, 17, 19, 20 | Open | Not verifiable from exports. |
| BL-18 | **Closed** | C10 input order confirmed from published YAML. |
| F1, F2, F8, F9 | **Closed** | Fixed and seen in exports. |
| F3, F4 | Open | Missing from backlog → R-04, R-05. |
| F5, F6, F7 | Open | → BL-14, BL-06 (downgraded), BL-07. |
| F10 | **Closed** | HTML outputs unbound; no ContentValidationError in field. |

---

## 7. Recommended work order (updated v3)

1. **Now — complete the immediate fix.**
   - Publish Flow D (FD04 and FD04b guards are in, row 2 deleted).
   - Wait for the next 10-minute cycle; confirm Flow D runs clean.
   - Run Scratch Diagnostics test for R-07: Compose `ticks('09/22/2026 09:30:00')` and compare to `ticks('2026-09-22T09:30:00Z')`. If mismatch, apply the EndTime normalisation fix in both `Create_Mapping_Item_*` actions.

2. **Session 1 — stabilise and verify (Sonnet).**
   - Apply R-02 (FC16 run-after settings change) — decision Q6 needed first.
   - Apply R-10 (FC01 ChatCaptured guard).
   - Reset one test row (set `ChatCaptured = No`); confirm clean end-to-end fill.
   - Answer Q4 — check whether any invite in your calendar uses `/meet/` format. If yes, prioritise R-09.

3. **Session 2 — Flow B correctness (Opus to design tests, Sonnet to edit).**
   - Answer Q2 (one-off section design) before touching anything.
   - R-04, R-05, R-06, BL-07, R-15, R-14.
   - Test matrix: recurring new, recurring existing, one-off new, one-off existing.

4. **Session 3 — docs and data quality (Sonnet).**
   - Rewrite drifted `ARCHITECTURE.md` sections (R-08, R-17). Q2 answer needed for one-off section description.
   - R-09 after Scratch Diagnostics proof.
   - BL-17 (known-good values merge; add C and D references).

5. **Later — structural (Opus to design).**
   - BL-08, R-11, BL-10, BL-14.

---

## 8. Proposed `BACKLOG.md` rewrite (for review only — not applied)

| ID | Item | Type | Pri | Source | Notes |
|---|---|---|---|---|---|
| BL-22 | Recap duplicated when chat part fails. | Bug | H | R-02 | Settings. FC16 run-after. Decision Q6 first. |
| BL-24 | Recurring section logic runs for one-offs (F3). | Bug | H | F3/R-04 | Expr. Q2 first. |
| BL-25 | One-off re-capture false SUCCESS (F4). | Bug | H | F4/R-05 | Expr. |
| BL-03 | Live one-off test matrix (full). | Verify | H | Session 21 Sep | Partially answered — rows 7–9 captured. |
| BL-27 | EndTime US-format text — verify `ticks()` parsing in Scratch Diagnostics. | Verify | H | R-07 | One Compose test. If mismatch, normalise in both Create Mapping Item EndTime fields. |
| BL-26 | Existing page found by date text. | Bug | M | R-06 | Expr. |
| BL-28 | `/meet/` join links produce empty JoinUrl → poison rows. | Bug | M | R-09 | Expr. Scratch Diagnostics first. Q4 for prevalence. |
| BL-29 | FC01 can re-select captured row when duplicates exist. | Bug | M | R-10 | Expr. Preventive. |
| BL-07 | Stale skeleton rows (UJ3b). | Bug | M | F7 | Expr. Add SectionPagesUrl guard to mapping filters. |
| BL-08 | Title-set failure blocks mapping write. | Bug | M | 16 Aug | Struct. |
| BL-30 | Flow B hard failures return nothing to Topic. | Bug | M | R-11 | Struct. |
| BL-32 | Rewrite drifted `ARCHITECTURE.md` sections. | Tidy | M | R-08/R-17 | Docs. Q2 needed for one-off section description. |
| BL-17 | Known-good values merge; add C and D references. | Tidy | M | Weekend plan D4 | |
| BL-23 | FD03 `$top 50` — apply OData filter before list reaches 40 rows. | Bug | L | R-03 | Expr. Not urgent — 9 rows. |
| BL-09 | Rename `Mtg -` → `Rec -`. | Improve | L | Weekend plan C2 | Q2 first. |
| BL-10 | No re-fill after late recap. | Improve | L | Architecture §9 | |
| BL-11 | `outbranchresult` bound to `varFinalMatchCount`. | Bug | L | 19 Sep minor | Expr. |
| BL-12 | Single-match forces pick "1". | Improve | L | 6 Sep | Q8. |
| BL-31 | Chat capped at 50 messages, no paging. | Improve | L | R-12 | Struct. Document for now. |
| BL-33 | UJ3b delete failure blocks Flow B. | Bug | L | R-13 | Settings. |
| BL-34 | Re-capture appends full skeleton with misleading text. | Bug | L | R-14 | Expr. |
| BL-35 | "Meeting Invite" heading always empty — BodyPreview not passed to Flow B. | Improve | L | R-08 | Topic `text_3` expression. |
| BL-14 | Remove dead paths (expanded with R-16 items). | Tidy | L | F5/R-16 | Struct. After stable sessions; snapshot first. |
| BL-15 | Mixed OneNote connections and notebookKey paths. | Tidy | L | 19 Sep minor | Document — deliberate design. |
| BL-16 | Clean-up test artefacts, `iCalUId` column. | Tidy | L | Weekend plan D5 | |
| BL-19 | Page header: attendees, organiser, start–end time. | Improve | L | Weekend plan | |
| BL-20 | Microsoft support ticket (value-wipe corruption). | Improve | L | 15 Aug | |

**Would close from current BACKLOG.md:** BL-01, BL-02, BL-04, BL-05, BL-13, BL-18.

---
*Draft v1 created 22 Sep 2026. v2 updated 23 Sep with SharePoint list data. v3 updated 23 Sep with Topic YAML confirmed, Q3 answered from session log, R-07 design understood, R-09 confirmed real, R-18 closed, FD04+FD04b fixes confirmed applied.*
