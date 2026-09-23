# End-to-end review — Flows A–D + Topic — 22 Sep 2026 — DRAFT v4

> **STATUS: DRAFT — all immediate fixes applied. Fill path stabilised. Remaining open items are Q2 (one-off section design) and Q4 (`/meet/` link prevalence). Review is substantially complete.**
>
> Updated 23 Sep 2026 (v4): R-07 closed — Scratch Diagnostics confirmed `ticks()` parses US-format EndTime correctly in this tenant. BL-27 closed. BACKLOG.md updated.
> - `ARCHITECTURE.md` has **not** been changed.

**Inputs reviewed:** published code-view exports of Flow A, B, C and D; Topic YAML (confirmed published); agent overview; flow overview pages; `ARCHITECTURE.md`; `BACKLOG.md`; `analysis-2026-09-19-end-to-end-review.md`; full SharePoint list export (10 rows, all columns); list Settings column-type page; `session-2026-09-18-endtime-joinurl-flowd.md`; Scratch Diagnostics run (23 Sep, ticks comparison).
**Not reviewed:** known-good-values references; run history detail; field-testing notes.
**Reviewer:** Claude (Opus 5.5), chat sessions 22–23 Sep 2026.

---

## 0. Open questions (updated 23 Sep v4)

### Answered

| # | Question | Answer | Affects |
|---|---|---|---|
| E3 | Row count and uncaptured rows | **9 rows (row 2 deleted). 1 uncaptured** (row 1, future 1:1). | R-01 |
| E3 | EndTime format | **US locale text `MM/dd/yyyy HH:mm:ss`** — intentional pipeline design; see R-07. | R-07 |
| E3 | ChatCaptured column type | **Yes/No** — filter expression working correctly. | — |
| E3 | Duplicate rows | **None.** | R-10 |
| Q1 | Is uploaded Topic YAML the published version? | **Yes — confirmed identical.** `text_7` = JoinUrl is live. No ContentValidationError reported. | R-08, BL-02, F10 |
| Q3 | Was FD03's today/yesterday filter ever built? | **Never built — only planned.** `session-2026-09-18` confirms FD03 was redesigned on 18 Sep with no date filter. `ARCHITECTURE.md` description is wrong. | R-03, R-17 |
| Scratch | Does `ticks()` parse US-format EndTime correctly? | **Yes — confirmed.** `ticks('09/22/2026 09:30:00')` = `ticks('2026-09-22T09:30:00Z')` = **639,256,662,000,000,000**. Identical. No normalisation fix needed. | R-07 |
| BL-01 | FD06d skipped? | **Superseded — confirmed.** FD06d fires; failure was FC05 on empty JoinUrl (row 2). | — |
| BL-02 | JoinUrl source | **Fixed.** Both Create Mapping Item actions read `text_7`; Topic sends JoinUrl there. | — |
| BL-04 | Offset value | **Fixed.** FD01 = 30. | — |
| BL-18 | C10 input order | **Fixed.** `text` = IsRecurring, `text_1` = Title, `text_2` = SeriesMasterId. | — |
| R-18 | Row 1 OccurrenceDate wrong format | **Closed — display artefact.** Stored value is `2026-10-02` (ISO). No edit needed. | — |

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

---

## SP. SharePoint list — full picture (23 Sep)

**9 rows (row 2 deleted 23 Sep). 1 uncaptured row remaining.**

| # | MeetingTitle | Type | OccurrenceDate | EndTime | JoinUrl | ChatCaptured | Notes |
|---|---|---|---|---|---|---|---|
| 1 | 1:1 David C \| Rich B | Recurring | 2026-10-02 | 10/02/2026 10:00 | Present | **FALSE** | Future meeting. EndTime US-format — confirmed to parse correctly in Power Automate (Scratch test 23 Sep). |
| ~~2~~ | ~~Mtg - Winning Together Peak call~~ | ~~One-off~~ | — | — | ~~**EMPTY**~~ | ~~FALSE~~ | **Deleted 23 Sep.** Root cause of fill-path failure. |
| 3 | SC&L FLT Stand-up | Recurring | 21/09/2026 | 09/21/2026 08:00:00 | Present | TRUE | |
| 4 | Supply Chain Tech and Data... | Recurring | 22/09/2026 | 09/22/2026 09:30:00 | Present | TRUE | |
| 5 | SC&L Portfolio Design Authority | Recurring | 22/09/2026 | 09/22/2026 10:00:00 | Present | TRUE | |
| 6 | SCT Programme Board | Recurring | 22/09/2026 | 09/22/2026 11:00:00 | Present | TRUE | |
| 7 | Mtg - Rapid Alerts | One-off | 22/09/2026 | 09/22/2026 15:25:00 | Present | TRUE | One-off fill confirmed working. |
| 8 | Mtg - Fortification | One-off | 22/09/2026 | 09/22/2026 12:30:00 | Present | TRUE | Same. |
| 9 | Mtg - TEST - JoinUrl Pipeline Check | One-off | 23/09/2026 | 09/23/2026 04:25:00 | Present | TRUE | Clean end-to-end test 23 Sep — confirmed working. |
| 10 | (1:1 Rich B — partial) | Recurring | — | — | — | TRUE | |

---

## 1. Summary (updated 23 Sep v4)

**Set-up path (Topic → A → B): healthy.** All recent runs succeeded; agent shows 100% successful runs over 7 days. One-off set-up and fill confirmed working (rows 7, 8, 9).

**Fill path (D → C): stabilised.**
- Row 2 (Winning Together Peak call, empty JoinUrl) was the sole cause of ~150 daily Flow D errors. Deleted 23 Sep.
- FD04 and FD04b updated with JoinUrl and PageSelfUrl guards. Flow D published. Clean run confirmed.
- EndTime US-format parsing verified in Scratch Diagnostics — `ticks()` parses correctly. No normalisation needed.
- Row 1 (future 1:1, 2 Oct) will enter FD04 when current; FD06c will hold on the False branch until EndTime + 30 min has elapsed.

**Two high-severity Flow B findings from 19 Sep (F3, F4) remain open** and are missing from `BACKLOG.md`. These are now in the updated backlog.

---

## 2. Findings (updated 23 Sep v4)

Risk key: **Expr** = expression or parameter edit only · **Settings** = run-after change · **Struct** = add, move or delete actions.

| ID | Component | Finding | Evidence | Impact | Sev | Fix | Risk | Confidence |
|---|---|---|---|---|---|---|---|---|
| R-01 | D / C | **FIXED.** Row 2 deleted; FD04 and FD04b JoinUrl + PageSelfUrl guards added; Flow D published; clean run confirmed. | SP list; peek code confirmed; Flow D run history 23 Sep | Was causing all ~150 daily errors | H | **Done** | — | **Confirmed + fixed** |
| R-02 | C | Recap duplicated when chat part fails. FC16 runs only if FC15 succeeds, but FC05t already appended the recap. | FC16 runAfter `FC15: Succeeded` | Recap re-appended on every retry cycle | H | FC16 run-after: Succeeded, Failed, Skipped, TimedOut | Settings | Logic verified; Q6 needed for trade-off |
| R-03 | D | FD03 `$top 50`, no filter or order. Today/yesterday filter was never built (Q3 confirmed). | FD03 code; 9 rows currently | Not urgent. Will bite once list exceeds 50 rows. | L | FD03 Filter Query `ChatCaptured eq 0`, Order By `ID desc` | Expr | **Confirmed** |
| R-04 | B | F3 still open: recurring section lookup/create runs for one-offs. | Condition expressions have no IsRecurring gate | Long one-off titles → two sections | H | F3 expressions | Expr | Verified; right fix depends on Q2 |
| R-05 | B | F4 still open: one-off re-capture can report SUCCESS falsely. | `Filter_Existing_Section_By_Name` uses 43-char name match | Nothing written; user told it saved | H | Filter by `pagesUrl` | Expr | Verified |
| R-06 | B | Existing page found by date substring. Fallback creates untitled page with no mapping update. | `Filter_Pages_By_Title`, `Guard_Create_Page_Fallback` | Wrong-page append; fails on empty `text_5` | M | Match by page id | Expr | Verified |
| R-07 | Topic → B → D | **CLOSED.** EndTime US-format is intentional (UTC\| pipeline design). Scratch Diagnostics 23 Sep confirmed `ticks('09/22/2026 09:30:00')` = `ticks('2026-09-22T09:30:00Z')` = 639,256,662,000,000,000. Power Automate parses US-format EndTime correctly in this tenant. No fix needed. | Scratch Diagnostics run 23 Sep | — | — | None required | — | **Confirmed closed** |
| R-08 | Topic → B | C10 contract confirmed from published YAML: `text_3` = HTML skeleton only; `text_7` = JoinUrl. `ARCHITECTURE.md` is wrong. | Topic C10 bindings | "Meeting Invite" heading always empty; docs misleading | M | Update `ARCHITECTURE.md`; optionally add BodyPreview (BL-35) | Docs | **Confirmed** |
| R-09 | A | Confirmed real from session log. `/meet/` format invites get empty JoinUrl → poison row pattern (same as row 2). `FA29D` was deliberately scoped to `/l/meetup-join/` on 18 Sep. | `session-2026-09-18` §2 | Those meetings become poison rows; blocked by new FD04 guard but never filled | M | Handle both formats in FA12B and FA29D–F | Expr | **Confirmed real. Q4 for prevalence.** |
| R-10 | C | FC01 (`$top 1`) can re-select an already-captured row when duplicates exist. | FC01 `$filter` | Endless re-append | M | Add `and ChatCaptured eq 0` | Expr | Preventive; no duplicates currently |
| R-11 | B | Hard failures return nothing to the Topic. No Scope/catch. | Tail action run-afters | Generic error; C12 unreachable; duplicate pages on retry | M | Scope + catch | Struct | Verified |
| R-12 | C | Chat limited to last 50 messages, no paging. | FC07 URI | Busy meetings truncated | L | Document; paging structural | Struct | Verified |
| R-13 | B | UJ3b delete failure blocks whole run. | IsRecurring runAfter `SUCCEEDED` only | One failed delete fails capture | L | Add Failed to run-after | Settings | Verified |
| R-14 | B | Re-capture appends full skeleton with misleading text. | `Compose_UpdateHtmlFragment` | Duplicate headings | L | Header + datestamp only | Expr | Verified |
| R-15 | B | `outbranchresult` returns `varFinalMatchCount`. | Respond body | Diagnostics only | L | Coalesce branch results | Expr | Verified |
| R-16 | A/B/Topic/C | Dead or literal logic (list below). | Exports | Confusion; corruption exposure | L | Leave until stable, then BL-14 | Struct | Verified |
| R-17 | Docs | `ARCHITECTURE.md` drift — confirmed items listed below. | Exports + session log vs doc | Future sessions reason from wrong design | M | Rewrite affected sections | Docs | Q2 still needed for one-off section description |

**R-16 dead or literal logic:** FA12 appends literal `"json(concat(...` (no `@`) to unused `varCandidates`; FA14 uses `item()` outside loop; FA15–FA26 never reached; FA28–FA30 single-match outputs always overwritten by C6D; Flow B D2 branch unreachable; `Compose_IgnoreSeriesMasterId` literal `''`; `outpagehtml`/`outupdatehtmlfragment` unbound; Topic C9B `PageTitle` unused; Flow C `AllowFallback` unused.

**R-17 confirmed drift:** Flow A uses V3 connector not V4; FD03 has no date filter (never built); Flow C appends to `body` not `#notes`/`#chat`; C10 contract (`text_3` HTML skeleton only, `text_7` = JoinUrl); Flow B and C use different notebookKey paths. **Pending Q2:** one-off section design.

---

## 3. Fixes applied — complete record

| Fix | Status | Date | Detail |
|---|---|---|---|
| Row 2 deleted | **Done** | 23 Sep | Winning Together Peak call removed from SP list |
| FD04b JoinUrl guard | **Done** | 23 Sep | `not(empty(coalesce(item()?['JoinUrl'], '')))` |
| FD04b PageSelfUrl guard | **Done** | 23 Sep | `not(empty(coalesce(item()?['PageSelfUrl'], '')))` |
| FD04 JoinUrl guard | **Done** | 23 Sep | Same addition to recurring filter |
| FD04 PageSelfUrl guard | **Done** | 23 Sep | Same addition to recurring filter |
| Flow D published | **Done** | 23 Sep | Clean run confirmed |
| R-07 normalisation fix | **Not needed** | 23 Sep | Scratch Diagnostics confirmed ticks() parses correctly |

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

**R-09** (prove in Scratch Diagnostics first — confirm `GetOnlineMeeting` accepts `/meet/` format):
```
@if(contains(coalesce(item()?['body'],''),'href="https://teams.microsoft.com/l/meetup-join/'), concat('https://teams.microsoft.com/l/meetup-join/', first(split(split(coalesce(item()?['body'],''),'href="https://teams.microsoft.com/l/meetup-join/')[1],'"'))), if(contains(coalesce(item()?['body'],''),'href="https://teams.microsoft.com/meet/'), concat('https://teams.microsoft.com/meet/', first(split(split(coalesce(item()?['body'],''),'href="https://teams.microsoft.com/meet/')[1],'"'))), ''))
```

**R-10:** FC01 `$filter` — append ` and ChatCaptured eq 0` to existing expression.

**R-14:** `Compose_UpdateHtmlFragment`: `@concat('<hr><p><em>Re-captured by Meeting Capture Agent on ', formatDateTime(utcNow(), 'd MMM yyyy HH:mm'), ' UTC.</em></p>')`

**R-15:** `outbranchresult`: `@{coalesce(outputs('Compose_Branch_Result'), outputs('Compose_Branch_Result_NoMatch'))}`

---

## 5. User journeys (updated v4)

| Journey | Recurring | One-off |
|---|---|---|
| New capture | OK | OK — confirmed working (rows 7, 8, 9) |
| Re-capture, same occurrence | Second skeleton appended (R-14); may pick wrong page (R-06) | False SUCCESS for long titles (R-05) |
| No-match day + navigation | OK | OK |
| Multi-match selection | OK | OK |
| Single match | Forced to pick "1" (BL-12) | Same |
| Auto fill, recap available | Fill path stabilised (R-01 fixed); duplication risk remains (R-02) | Working for valid JoinUrl |
| Auto fill, no recap | Chat only — correct design | Same |
| No Teams link / empty JoinUrl | Blocked by FD04 guard | Same |
| `/meet/` format invite | Empty JoinUrl → blocked by guard; no fill until R-09 applied | Same |
| Past-date capture | Set-up OK; fill OK within 50-row limit | Same |
| Future meeting (row 1, 2 Oct 1:1) | Set-up done; fill will fire at EndTime + 30 min — timing confirmed correct | — |

---

## 6. Backlog reconciliation (final for this review)

| ID | Status | Reason |
|---|---|---|
| BL-01 | **Closed** | FD06d fires; root cause was empty JoinUrl on row 2. Row 2 deleted. |
| BL-02 | **Closed** | `text_7` = JoinUrl confirmed in published YAML and both Create Mapping Item actions. |
| BL-03 | Partially answered | One-off set-up and fill confirmed working (rows 7–9). Full test matrix still needed. |
| BL-04 | **Closed** | FD01 = 30 confirmed. |
| BL-05 | **Closed** | Flow C appends to `body`; no slots; no AllowFallback gate. Docs need updating (R-17). |
| BL-06 | **Downgraded to L** | 9 rows — safe for months. |
| BL-07 | Open | No SectionPagesUrl guard in Flow B mapping filters. |
| BL-08 | Open | OF09 gate depends on Set_PageTitle succeeding. |
| BL-09 | Open | Still `Mtg -`. Tied to Q2. |
| BL-10 | Open | Design gap. |
| BL-11 | Changed | UpdatedAppend fixed; `outbranchresult` remains (R-15); formatDateTime guard → R-06. |
| BL-12 | Open | Q8. |
| BL-13 | **Closed** | meetup-join links work; `/meet/` tracked as R-09. |
| BL-14 | Open, expanded | Add R-16 items. |
| BL-15 | Open | Document. |
| BL-16, 17, 19, 20 | Open | Not verifiable from exports. |
| BL-18 | **Closed** | C10 input order confirmed from published YAML. |
| BL-27 | **Closed** | Scratch Diagnostics 23 Sep confirmed `ticks()` parses US-format EndTime correctly. |
| F1, F2, F8, F9 | **Closed** | Fixed and seen in exports. |
| F3, F4 | Open | Missing from old backlog → now BL-24, BL-25. |
| F5, F6, F7 | Open | → BL-14, BL-06 (downgraded), BL-07. |
| F10 | **Closed** | No ContentValidationError in field. |

---

## 7. Recommended work order (updated v4)

1. **Next — stabilise fills and apply quick wins (Sonnet).**
   - Decide Q6 (recap duplication trade-off), then apply R-02 (FC16 run-after).
   - Apply R-10 (FC01 ChatCaptured guard).
   - Reset one test row (`ChatCaptured = No`); confirm clean end-to-end fill.
   - Answer Q4 (any `/meet/` format invites?). If yes, prioritise R-09.

2. **Session 2 — Flow B correctness (Opus to design, Sonnet to edit).**
   - Answer Q2 (one-off section design) first.
   - R-04, R-05, R-06, BL-07, R-15, R-14.
   - Test matrix: recurring new, recurring existing, one-off new, one-off existing.

3. **Session 3 — docs and data quality (Sonnet).**
   - Rewrite `ARCHITECTURE.md` (R-08, R-17). Q2 needed for one-off section description.
   - R-09 after Scratch Diagnostics proof (Q4 first).
   - BL-17 (known-good values merge; add C and D references).

4. **Later — structural (Opus to design).**
   - BL-08, R-11, BL-10, BL-14.

---
*Draft v1 created 22 Sep 2026. v2: SharePoint list data. v3: Topic YAML confirmed, Q3 answered, R-09 confirmed, R-18 closed, FD04+FD04b fixes applied. v4: R-07 closed (Scratch Diagnostics 23 Sep), BL-27 closed, BACKLOG.md updated.*
