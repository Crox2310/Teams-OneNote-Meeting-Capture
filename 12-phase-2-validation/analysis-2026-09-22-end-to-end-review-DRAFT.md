# End-to-end review — Flows A–D + Topic — 22 Sep 2026 — DRAFT

> **STATUS: DRAFT — REQUIRES FURTHER WORK. DO NOT ACT ON THE FINDINGS YET.**
>
> This review was produced from code-view exports only. Run-history detail, SharePoint list data, field-testing notes and the known-good-values references were **not** reviewed. Several findings — including the diagnosis of why the fill path is failing — are inferred and could be wrong.
>
> - `BACKLOG.md` has **not** been changed. The proposed rewrite is at the end of this file for review only.
> - `ARCHITECTURE.md` has **not** been changed.
> - Next step: David supplies the evidence and answers in **§0**, then the review is re-run (Opus) and finalised.

**Inputs reviewed:** published code-view exports of Flow A, B, C and D (per-action Peek code plus designer screenshots); Topic YAML (Meeting Capture v4 rebuild); agent overview (Teams → OneNote Meeting Capture, GPT-4.1, published 21 Sep); flow overview pages (status, connections, recent run list); `ARCHITECTURE.md`, `BACKLOG.md`, `analysis-2026-09-19-end-to-end-review.md`.
**Not reviewed:** known-good-values references (master, Flow A, 18 Sep addendum); any run detail; SharePoint list contents; field-testing notes.
**Reviewer:** Claude (Opus 5.5), chat session 22 Sep 2026, late evening.

---

## 0. Open questions and evidence needed (from David)

These determine whether the findings below are right and what order to fix them in. Each item names the finding it affects.

### Evidence to capture

| # | What | Why it matters | Affects |
|---|---|---|---|
| E1 | One **failed Flow C run** from 22 Sep 23:20–23:22: the failing action, its error message, and the trigger inputs (`text`, `text_1`, `text_2`, `text_4`) | The diagnosis that the ~350 ms failures are `FC00a startOfDay()` on an empty or invalid OccurrenceDate is inferred from timing only. If the failure is elsewhere, the R-01 fix is wrong. | R-01, BL-01 |
| E2 | One **Flow D run** from the same window: FD04, FD04b and FD04c outputs (row count and IDs), and each FD06 iteration's FD06c inputs and result | Confirms which rows are looping and settles the old BL-01 "True branch but FD06d skipped" observation. | R-01, R-03, BL-01 |
| E3 | **SharePoint list**: total row count, plus an export of every row with `ChatCaptured = No` (ID, MeetingTitle, SeriesMasterId, MeetingId, OccurrenceDate, EndTime, JoinUrl, SectionPagesUrl, PageSelfUrl) | Row count decides whether FD03's `$top 50` is already hiding rows. The export shows what the poison rows actually look like and whether duplicate rows exist. | R-01, R-03, R-07, R-10, BL-06 |
| E4 | One **recurring** and one **one-off** Flow B run: raw trigger inputs (especially `text_6` EndTime and `text_7` JoinUrl) and the `OutSPItemCount` output | Confirms whether EndTime arrives as ISO or US-formatted text, and whether the join URL arrives in `text_7`. | R-07, R-08, BL-02 |
| E5 | Any **successful Flow C run** since 20 Sep, and the page it wrote to | Tells us whether the fill path has ever worked in field testing, or has been failing throughout. | Summary, R-02 |
| E6 | A look at recently filled **OneNote pages** — any with the recap or chat appended more than once? | Tests whether R-02 (duplicate recap) or R-10 (duplicate rows) has already happened in practice. | R-02, R-10 |
| E7 | **Field-testing notes** so far: meetings captured, what went wrong, what looked right | Currently empty in the intake table; may contain issues this review hasn't seen. | All |

### Questions only David can answer

| # | Question | Affects |
|---|---|---|
| Q1 | Is the uploaded Topic YAML the **published** version? Has `ContentValidationError` appeared since 20 Sep? (`ARCHITECTURE.md` says the join URL must travel in `text_3`; the export sends it in `text_7`.) | R-08, BL-02, F10 |
| Q2 | **One-off section design** — is the intended design a single fixed "One-Off Meetings" section (as in `ARCHITECTURE.md` and the weekend plan), or per-title `Mtg -` sections (as in the export)? Was the fixed-section change never made, or deliberately reversed? | R-04, R-05, R-17, BL-09 |
| Q3 | Was FD03's **today/yesterday filter** ever built and then lost (e.g. in a value wipe), or was it only planned? | R-03, R-17 |
| Q4 | Do any of your meetings use the newer **`teams.microsoft.com/meet/`** join-link format? | R-09 |
| Q5 | Is **Flow D** still running every 10 minutes, or has it been paused? Do you want it paused until the fill path is fixed? | R-01 |
| Q6 | For a failed chat step, is it acceptable to **mark the row captured anyway** (no duplicates, but the chat is lost), or would you rather keep retrying and accept duplicate recaps until fixed? | R-02 |
| Q7 | Should I read the **known-good-values references** before finalising? The corruption check in this draft is based on the exports alone. | Corruption check |
| Q8 | Is the **Flow A single-match** behaviour (forced to pick "1") a problem in the field, or acceptable? | BL-12 |

---

## 1. Summary (provisional)

**Set-up path (Topic → A → B): healthy.** Recent Flow A and B runs succeeded; the agent shows 100% successful runs over 7 days.

**Fill path (D → C): appears to be failing.**
- Flow D's error-trend panel shows ~144–152 errors a day — about one per 10-minute run.
- Flow C's last six runs failed in 307–424 ms within three minutes (22 Sep 23:20–23:22) — consistent with one Flow D run looping over six rows.
- That speed is too fast for any connector call, so the failure is probably an expression. **Inferred** cause: `FC00a startOfDay(text_2)` on an empty or invalid OccurrenceDate. Needs E1.
- Rows that can never succeed are retried every 10 minutes indefinitely.
- There is also a path where the recap gets appended repeatedly (R-02).

**BL-01 looks superseded** — FD06d does fire; the failures are inside Flow C. Needs E2 to confirm.

**Docs have drifted from the build** (see R-17), and two high-severity Flow B findings from 19 Sep (F3, F4) are still open in the exports but missing from `BACKLOG.md`.

---

## 2. Findings (provisional)

Risk key: **Expr** = expression or parameter edit only · **Settings** = run-after change · **Struct** = add, move or delete actions.

| ID | Component | Finding | Evidence | Impact | Sev | Fix | Risk | Confidence |
|---|---|---|---|---|---|---|---|---|
| R-01 | D / C | Rows that can never succeed are retried forever. FD04/FD04b don't check OccurrenceDate window, JoinUrl or PageSelfUrl. | FD04/FD04b `where`; D error trend ~150/day; C failures 307–424 ms | Every D run fails; real fills buried; wasted C runs | H | Tighten FD04 and FD04b | Expr | Cause inferred — needs E1–E3 |
| R-02 | C | Recap duplicated when the chat part fails. FC16 runs only if FC15 succeeds, but FC05t already appended the recap. | FC16 runAfter `FC15: Succeeded`; FC06 runAfter FC05c on any status | Recap re-appended every 10 min until fixed | H | FC16 run-after on FC15: Succeeded, Failed, Skipped, TimedOut | Settings | Logic verified; occurrence needs E6. Trade-off needs Q6 |
| R-03 | D | FD03 `$top 50`, no filter or order — past 50 rows the scheduler only sees the oldest 50. `ARCHITECTURE.md` says there's a today/yesterday filter; the export has none. | FD03 code; row IDs ≥ 429 | New meetings never filled | H (latent) | FD03 Filter Query `ChatCaptured eq 0`, Order By `ID desc` | Expr | Severity depends on E3; history on Q3 |
| R-04 | B | F3 still open: recurring section lookup/create runs for one-offs. FB-F01 caps at 37 chars, `Compose_SafeSectionName` at 43. | `Condition_Section_Exists_Recurring` and `Condition_Section_Count_Is_Zero` have no IsRecurring gate | Long one-off titles create two sections; short titles risk duplicate-section race | H | F3 expressions | Expr | Verified in export; right fix depends on Q2 |
| R-05 | B | F4 still open: one-off re-capture can report SUCCESS falsely. Existing-page branch finds section by 43-char name. | `Filter_Existing_Section_By_Name` uses `Compose_SafeSectionName_ExistingBranch` | Nothing written; user told it saved | H | Filter by `pagesUrl`; source section from either mapping | Expr | Verified in export |
| R-06 | B | Existing page found by date substring. Fallback creates an untitled page with no mapping update. | `Filter_Pages_By_Title`, `Guard_Create_Page_Fallback` | Wrong-page append in shared sections; fails on empty `text_5` | M | Match by page id | Expr | Verified in export |
| R-07 | Topic → B → D | EndTime not stored as UTC ISO. Row 429 = `09/22/2026 09:30:00`. | 22 Sep session; C6D; `Create_Mapping_Item_*` | Culture-dependent parse in FD06c; possible 1-hour late fill in BST | M | Verify raw `text_6`, then normalise at write | Expr | Inferred — needs E4 |
| R-08 | Topic → B | C10 contract differs from `ARCHITECTURE.md`: `text_3` is an HTML skeleton (no `\|\|\|\|`, no invite body, no `data-id` divs); `text_7` = `Topic.OnlineMeetingUrl`. | Topic C10 bindings | Docs describe a non-live build; "Meeting Invite" always empty | M | Update docs; optionally add BodyPreview | Docs | Depends on Q1 |
| R-09 | A | Join link extraction only matches `/l/meetup-join/`; `/meet/` invites get empty JoinUrl. | FA12B `OnlineMeetingUrl` | Those meetings never filled | M | Handle both formats (prove in Scratch Diagnostics first) | Expr | Prevalence inferred — Q4 |
| R-10 | C | FC01 (`$top 1`) can re-select an already-captured row when duplicates exist. | FC01 `$filter` | Endless re-append with duplicate rows | M | Add `and ChatCaptured eq 0` | Expr | Logic verified; occurrence needs E3/E6 |
| R-11 | B | Hard failures return nothing to the Topic (no Scope/catch). | Tail actions run after Succeeded only | Generic error; C12 retry unreachable; duplicate pages on retry | M | Scope + catch | Struct | Verified in export |
| R-12 | C | Chat limited to last 50 messages, no paging. | FC07 URI | Busy meetings truncated | L | Document; paging is structural | Struct | Verified |
| R-13 | B | UJ3b delete failure blocks the whole run. | IsRecurring runAfter `UJ3b_Delete_Stale_Rows: SUCCEEDED` | One failed delete fails the capture | L | Add Failed to run-after | Settings | Verified |
| R-14 | B | Re-capture appends a second full skeleton with misleading "preserved below" text. | `Compose_UpdateHtmlFragment` | Duplicate headings | L | Header + note only | Expr | Verified |
| R-15 | B | `outbranchresult` returns `varFinalMatchCount`. | Respond body | Diagnostics only | L | Coalesce branch results | Expr | Verified |
| R-16 | A / B / Topic / C | Dead or literal logic (list below). | Exports | Confusion; corruption exposure | L | Leave until stable, then BL-14 | Struct | Verified |
| R-17 | Docs | `ARCHITECTURE.md` drift (list below). | Exports vs doc | Future sessions reason from the wrong design | M | Rewrite affected sections | Docs | Depends on Q1–Q3 |

**R-16 dead or literal logic:** FA12 appends a literal `"json(concat(...` (no `@`) to unused `varCandidates`; FA14 uses `item()` outside a loop and is unused; FA15–FA26 never reached (Topic always sends `NONE`); FA28–FA30 single-match outputs always overwritten by C6D (incl. FA28C `endWithTimeZone`); Flow B D2 branch unreachable; `Compose_IgnoreSeriesMasterId` is literal `''`; `outpagehtml` / `outupdatehtmlfragment` returned but unbound; Topic C9B `PageTitle` unused; Flow C `AllowFallback` unused.

**R-17 `ARCHITECTURE.md` drift:** Flow A uses the V3 calendar connector, not V4; one-offs get per-title `Mtg -` sections, not a fixed section; FD03 has no date filter; Flow C appends to `body`, not `#notes`/`#chat`; C10 contract (R-08); Flow B and C use different `notebookKey` paths (works because `sectionId` is an absolute URL — inferred).

### Candidate fix expressions (not yet approved)

**R-01 — FD04 `where`** (FD04b takes the same three additions):
```
@and(not(equals(item()?['ChatCaptured'], true)), not(empty(coalesce(item()?['SeriesMasterId'], ''))), not(empty(coalesce(item()?['EndTime'], ''))), not(empty(coalesce(item()?['JoinUrl'], ''))), not(empty(coalesce(item()?['PageSelfUrl'], ''))), greaterOrEquals(coalesce(item()?['OccurrenceDate'], ''), formatDateTime(addDays(utcNow(), -2), 'yyyy-MM-dd')))
```

**R-04 (F3)**
- `Condition_Section_Exists_Recurring`: `@and(equals(toLower(string(triggerBody()?['text'])), 'true'), equals(outputs('Compose_Section_Match_Count_Recurring'), 1))` equals `true`
- `Condition_Section_Count_Is_Zero`: `@and(equals(toLower(string(triggerBody()?['text'])), 'true'), equals(outputs('Compose_Section_Match_Count_Recurring'), 0))` equals `true`

**R-05 (F4)**
- `Condition_Recurring_TargetSection`: `@not(empty(first(coalesce(body('Filter_Existing_Mapping'), body('OF01_—_Filter_Existing_Mapping_OneOff'), createArray()))?['SectionPagesUrl']))` equals `true`
- `Set_varTargetSectionPagesUrl_ExistingMapping`: `@first(coalesce(body('Filter_Existing_Mapping'), body('OF01_—_Filter_Existing_Mapping_OneOff'), createArray()))?['SectionPagesUrl']`
- `Filter_Existing_Section_By_Name` where: `@equals(item()?['pagesUrl'], variables('varTargetSectionPagesUrl'))`

**R-06 — `Filter_Pages_By_Title` where:** `@equals(item()?['id'], outputs('Compose_ExistingPageId'))`

**R-07 — EndTime in both `Create_Mapping_Item_*`** (only after E4):
`@if(empty(coalesce(triggerBody()?['text_6'], '')), '', formatDateTime(triggerBody()?['text_6'], 'yyyy-MM-ddTHH:mm:ssZ'))`

**R-09 — FA12B `OnlineMeetingUrl`:**
```
@if(contains(coalesce(item()?['body'],''),'href="https://teams.microsoft.com/l/meetup-join/'), concat('https://teams.microsoft.com/l/meetup-join/', first(split(split(coalesce(item()?['body'],''),'href="https://teams.microsoft.com/l/meetup-join/')[1],'"'))), if(contains(coalesce(item()?['body'],''),'href="https://teams.microsoft.com/meet/'), concat('https://teams.microsoft.com/meet/', first(split(split(coalesce(item()?['body'],''),'href="https://teams.microsoft.com/meet/')[1],'"'))), ''))
```

**R-10 — FC01 `$filter`:** wrap the existing expression: `@concat(<existing expression>, ' and ChatCaptured eq 0')`

**R-14 — `Compose_UpdateHtmlFragment`:** `@concat('<hr><p><em>Re-captured by Meeting Capture Agent on ', formatDateTime(utcNow(), 'd MMM yyyy HH:mm'), ' UTC.</em></p>')`

**R-15 — `outbranchresult`:** `@{coalesce(outputs('Compose_Branch_Result'), outputs('Compose_Branch_Result_NoMatch'))}`

### User journeys (from exports only)

| Journey | Recurring | One-off |
|---|---|---|
| New capture | OK | Works; page may land in a 43-char section (R-04) |
| Re-capture, same occurrence | Second skeleton appended (R-14); may pick wrong page (R-06) | False SUCCESS for titles > 37 chars (R-05) |
| No-match day + navigation | OK | OK |
| Multi-match selection | OK | OK |
| Single match | Forced to pick "1" (BL-12) | Same |
| Auto fill, recap available | Currently failing (R-01); duplication risk (R-02) | Never proven live (BL-03) |
| Auto fill, no recap | Chat only — correct | Same |
| No Teams link | Poison row (R-01); also `/meet/` links (R-09) | Same |
| Past-date capture | Set-up OK; fill skipped once > 2 days old (after R-01) | Same |

---

## 3. Backlog reconciliation (provisional)

| ID | Status | Reason |
|---|---|---|
| BL-01 | Superseded? | FD06d fires; failures are in Flow C. Confirm with E2. |
| BL-02 | Fixed? | Both Create Mapping Item actions read `text_7`; Topic sends `OnlineMeetingUrl` there. Confirm with E4 and Q1. |
| BL-03 | Open | No live one-off fill evidence. |
| BL-04 | Fixed | FD01 = 30. |
| BL-05 | Superseded | Flow C appends to `body`; no AllowFallback gate; no duplicate headings. Docs are wrong (R-17). |
| BL-06 | Open | `$top 500` unchanged. |
| BL-07 | Open | No `SectionPagesUrl` guard. |
| BL-08 | Open | OF09 gate still depends on `Set_PageTitle` succeeding. |
| BL-09 | Open | Still `Mtg -`. Tied to Q2. |
| BL-10 | Open | Design gap. |
| BL-11 | Changed | UpdatedAppend fixed; `outbranchresult` remains (R-15); `formatDateTime` guard covered by R-06. |
| BL-12 | Open | No MatchCount = 1 branch in Topic. Reclassify as Improve (Q8). |
| BL-13 | Fixed, caveat | FA12B carries join link for meetup-join invites; `/meet/` → R-09. |
| BL-14 | Open, expanded | Add R-16 items. |
| BL-15 | Open | Document. |
| BL-16, 17, 19, 20 | Open | Not verifiable from exports. |
| BL-18 | Fixed | `text` = IsRecurring, `text_1` = Title, `text_2` = SeriesMasterId. |
| F1, F2, F8, F9 | Fixed | Seen in exports. |
| F3, F4 | Open | Missing from backlog → R-04, R-05. |
| F5, F6, F7 | Open | → BL-14, BL-06, BL-07. |
| F10 | Unconfirmed, low | HTML outputs still returned but unbound. |

---

## 4. Provisional work order

1. **Session 1 — evidence and answers (Opus, no edits).** Collect E1–E7, answer Q1–Q8, pause Flow D if poison rows are confirmed. Re-run this review and finalise.
2. **Session 2 — stabilise fill path (Sonnet).** R-01, R-10 (Expr); R-03 (parameter); R-02 (Settings, per Q6). Publish; reset one test row; confirm one clean fill.
3. **Session 3 — Flow B correctness (Opus to design tests, Sonnet to edit).** R-04, R-05 (per Q2), R-06, BL-07, R-15, R-14. Test matrix incl. a long-titled one-off.
4. **Session 4 — contract and data quality (Sonnet).** R-09 (after Scratch Diagnostics proof), R-07 (after E4), rewrite `ARCHITECTURE.md` (R-17), BL-17.
5. **Later — structural (Opus to design).** BL-08, R-11, BL-06, BL-10, BL-14.

---

## 5. Proposed `BACKLOG.md` rewrite (for review only — not applied)

New items BL-21 onwards come from this review; existing IDs are kept where still open.

| ID | Item | Type | Pri | Source | Notes |
|---|---|---|---|---|---|
| BL-21 | Poison rows retried forever (FD04/FD04b gaps) | Bug | H | R-01 | Expr. Confirm cause first (E1). |
| BL-22 | Recap duplicated when chat part fails | Bug | H | R-02 | Settings. Decision Q6. |
| BL-23 | FD03 `$top 50`, no filter or order | Bug | H | R-03 | Expr. Check row count. |
| BL-24 | Recurring section logic runs for one-offs (F3) | Bug | H | F3 / R-04 | Expr. Depends on Q2. |
| BL-25 | One-off re-capture false SUCCESS (F4) | Bug | H | F4 / R-05 | Expr. |
| BL-03 | Live one-off end-to-end test | Verify | H | Session 21 Sep | After BL-21–25. |
| BL-26 | Existing page found by date text | Bug | M | R-06 | Expr. |
| BL-27 | EndTime not stored as UTC ISO | Bug | M | R-07 | Verify first (E4). |
| BL-28 | `/meet/` join links not recognised | Bug | M | R-09 | Expr. Prove in Scratch Diagnostics. |
| BL-29 | FC01 can re-select captured row | Bug | M | R-10 | Expr. |
| BL-06 | Flow B `Get_items` `$top 500` | Bug | M | F6 | Source-side filter. |
| BL-07 | Stale skeleton rows (UJ3b) | Bug | M | F7 | Expr. |
| BL-08 | Title-set failure blocks mapping write | Bug | M | 16 Aug | Struct. |
| BL-30 | Flow B hard failures return nothing | Bug | M | R-11 | Struct. |
| BL-32 | Rewrite drifted `ARCHITECTURE.md` sections | Tidy | M | R-08 / R-17 | Docs. |
| BL-17 | Known-good values merge; add C and D | Tidy | M | Weekend plan D4 | |
| BL-09 | Rename `Mtg -` → `Rec -` | Improve | L | Weekend plan C2 | With Q2. |
| BL-10 | No re-fill after late recap | Improve | L | Architecture §9 | |
| BL-11 | `outbranchresult` binding | Bug | L | 19 Sep minor | Expr. |
| BL-12 | Single-match forces pick "1" | Improve | L | 6 Sep | Q8. |
| BL-31 | Chat capped at 50 messages | Improve | L | R-12 | Struct. |
| BL-33 | UJ3b delete failure blocks Flow B | Bug | L | R-13 | Settings. |
| BL-34 | Re-capture appends full skeleton | Bug | L | R-14 | Expr. |
| BL-35 | "Meeting Invite" heading always empty | Improve | L | R-08 | Topic expression. |
| BL-14 | Remove dead paths (expanded) | Tidy | L | F5 / R-16 | Struct, after stable sessions. |
| BL-15 | Mixed OneNote connections / notebookKey | Tidy | L | 19 Sep minor | Document. |
| BL-16 | Clean-up test artefacts, `iCalUId` | Tidy | L | Weekend plan D5 | |
| BL-19 | Page header: attendees, organiser, times | Improve | L | Weekend plan | |
| BL-20 | Microsoft support ticket | Improve | L | 15 Aug | |

**Would close:** BL-01 (superseded, pending E2), BL-02 (pending E4/Q1), BL-04, BL-05, BL-13, BL-18.

---
*Draft created 22 September 2026. To be finalised after §0 is answered.*
