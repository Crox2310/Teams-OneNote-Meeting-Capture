# End-to-end review — Flows A–D + Topic — 22 Sep 2026 — FINAL

> **STATUS: REVIEW COMPLETE.** All questions answered. Immediate fixes applied and verified. Remaining findings are in BACKLOG.md.
>
> Updated 23 Sep 2026 (v5 — final): Q2, Q4, Q6, Q8 answered; R-02 fix description corrected per Q6 decision; R-04/R-05/R-17 updated with `Evt Mtg -` / `Mtg -` duplicate section detail; new BL-36 added; work order finalised.
> - `ARCHITECTURE.md` has **not** been changed — see BL-32.

**Inputs reviewed:** published code-view exports of Flow A, B, C and D; Topic YAML (confirmed published); agent overview; flow overview pages; `ARCHITECTURE.md`; `BACKLOG.md`; `analysis-2026-09-19-end-to-end-review.md`; full SharePoint list export (10 rows, all columns); list Settings column-type page; `session-2026-09-18-endtime-joinurl-flowd.md`; Scratch Diagnostics run (23 Sep, ticks comparison).
**Not reviewed:** known-good-values references; run history detail; field-testing notes.
**Reviewer:** Claude (Opus 5.5 / Sonnet 4.6), chat sessions 22–23 Sep 2026.

---

## 0. Questions — all answered (23 Sep)

| # | Question | Answer | Affects |
|---|---|---|---|
| E3 | Row count and uncaptured rows | **9 rows (row 2 deleted). 1 uncaptured** (row 1, future 1:1). | R-01 |
| E3 | EndTime format | **US locale text `MM/dd/yyyy HH:mm:ss`** — intentional pipeline design. | R-07 |
| E3 | ChatCaptured column type | **Yes/No** — filter expression working correctly. | — |
| E3 | Duplicate rows | **None.** | R-10 |
| Q1 | Is uploaded Topic YAML the published version? | **Yes.** `text_7` = JoinUrl is live. No ContentValidationError reported. | R-08, BL-02, F10 |
| Q2 | One-off section design — fixed section or per-title? | **Per-title `Evt Mtg -` is the intended prefix.** The fix was never completed — the old `Mtg -` code path still fires alongside `Evt Mtg -`, creating two sections per one-off capture. → BL-36. | R-04, R-05, R-17, BL-09, BL-36 |
| Q3 | Was FD03's today/yesterday filter ever built? | **Never built — only planned.** `ARCHITECTURE.md` description is wrong. | R-03, R-17 |
| Q4 | Do any meetings use `/meet/` join-link format? | **No — all meetings use standard `/l/meetup-join/` format** (confirmed from Outlook invite). R-09 stays at M priority. | R-09, BL-28 |
| Q6 | For a failed chat step: mark captured or keep retrying? | **`ChatCaptured` should only be true when content was actually written.** If either recap (FC05t) or chat (FC15) fails, leave the row uncaptured and let Flow D retry. Duplicate recaps on retry are acceptable; silent content loss is not. | R-02, BL-22 |
| Q7 | Read known-good-values references before finalising? | **Not needed** — SP list and exports provided sufficient evidence. | — |
| Q8 | Is single-match forced pick "1" a problem in the field? | **No — acceptable in practice.** BL-12 stays at L. | BL-12 |
| Scratch | Does `ticks()` parse US-format EndTime correctly? | **Yes.** `ticks('09/22/2026 09:30:00')` = `ticks('2026-09-22T09:30:00Z')` = **639,256,662,000,000,000**. No normalisation fix needed. | R-07 |
| BL-01 | FD06d skipped? | **Superseded.** FD06d fires; failure was FC05 on empty JoinUrl (row 2). | — |
| BL-02 | JoinUrl source | **Fixed.** Both Create Mapping Item actions read `text_7`. | — |
| BL-04 | Offset value | **Fixed.** FD01 = 30. | — |
| BL-18 | C10 input order | **Fixed.** `text` = IsRecurring, `text_1` = Title, `text_2` = SeriesMasterId. | — |
| E5 | Successful Flow C run since 20 Sep? | **Yes** — rows 7, 8, 9 all ChatCaptured = TRUE. | — |
| R-18 | Row 1 OccurrenceDate wrong format | **Closed — display artefact.** Stored as `2026-10-02` (ISO). No edit needed. | — |

---

## SP. SharePoint list — full picture (23 Sep)

**9 rows (row 2 deleted 23 Sep). 1 uncaptured row remaining.**

| # | MeetingTitle | Type | OccurrenceDate | EndTime | JoinUrl | ChatCaptured | Notes |
|---|---|---|---|---|---|---|---|
| 1 | 1:1 David C \| Rich B | Recurring | 2026-10-02 | 10/02/2026 10:00 | Present | **FALSE** | Future meeting. EndTime parses correctly (Scratch test 23 Sep). |
| ~~2~~ | ~~Mtg - Winning Together Peak call~~ | ~~One-off~~ | — | — | ~~EMPTY~~ | ~~FALSE~~ | **Deleted 23 Sep.** Root cause of fill-path failure. |
| 3 | SC&L FLT Stand-up | Recurring | 21/09/2026 | 09/21/2026 08:00:00 | Present | TRUE | |
| 4 | Supply Chain Tech and Data... | Recurring | 22/09/2026 | 09/22/2026 09:30:00 | Present | TRUE | |
| 5 | SC&L Portfolio Design Authority | Recurring | 22/09/2026 | 09/22/2026 10:00:00 | Present | TRUE | |
| 6 | SCT Programme Board | Recurring | 22/09/2026 | 09/22/2026 11:00:00 | Present | TRUE | |
| 7 | Mtg - Rapid Alerts | One-off | 22/09/2026 | 09/22/2026 15:25:00 | Present | TRUE | One-off fill confirmed working. |
| 8 | Mtg - Fortification | One-off | 22/09/2026 | 09/22/2026 12:30:00 | Present | TRUE | Same. |
| 9 | Mtg - TEST - JoinUrl Pipeline Check | One-off | 23/09/2026 | 09/23/2026 04:25:00 | Present | TRUE | Clean end-to-end test 23 Sep. |
| 10 | (1:1 Rich B — partial) | Recurring | — | — | — | TRUE | |

---

## 1. Summary

**Set-up path (Topic → A → B): healthy.** All runs succeeded. One-off set-up and fill confirmed working (rows 7, 8, 9).

**Fill path (D → C): stabilised.** Row 2 (empty JoinUrl) deleted. FD04 and FD04b hardened with JoinUrl and PageSelfUrl guards. Flow D published. Clean run confirmed. EndTime parsing verified.

**Known remaining bugs:** duplicate `Evt Mtg -` / `Mtg -` sections on one-off capture (BL-36, H); recap duplication on chat failure (BL-22, H); Flow B one-off re-capture false SUCCESS (BL-25, H); recurring section logic running for one-offs (BL-24, H). None block daily use for the current meeting set.

---

## 2. Findings (final)

Risk key: **Expr** = expression or parameter edit only · **Settings** = run-after change · **Struct** = add, move or delete actions.

| ID | Component | Finding | Evidence | Impact | Sev | Fix | Risk | Status |
|---|---|---|---|---|---|---|---|---|
| R-01 | D / C | **FIXED.** Row 2 deleted; FD04 and FD04b guards added; Flow D published; clean run confirmed. | SP list; peek code; run history 23 Sep | Was ~150 daily errors | H | Done | — | **Closed** |
| R-02 | C | FC16 (`Update_Item_ChatCaptured`) should run only after **both** FC05t (recap) and FC15 (chat) succeed. Currently FC16 runs after FC15: Succeeded only, but FC05t has already appended the recap — so if FC15 fails, the next retry re-appends the recap with no chat. **Q6 decision: leave ChatCaptured false if either step fails; accept duplicate recaps on retry rather than marking captured with missing content.** Fix: FC16 run-after on FC05t: Succeeded AND FC15: Succeeded. | FC16 runAfter; Q6 answer | Recap re-appended on every retry when chat fails | H | FC16 run-after: require both FC05t and FC15 Succeeded | Settings | Open — BL-22 |
| R-03 | D | FD03 `$top 50`, no filter. Today/yesterday filter never built (Q3). | FD03 code; 9 rows | Not urgent — 9 rows. | L | OData filter + `ID desc` order | Expr | Open — BL-23 |
| R-04 | B | **Updated (Q2).** The `Evt Mtg -` prefix path was partially built but the old `Mtg -` code path was never removed. Both fire on one-off capture — two sections created. Condition expressions also have no IsRecurring gate so recurring section logic runs for one-offs too. | Flow B exports; Q2 answer | Two sections per one-off capture | H | Remove `Mtg -` path; add IsRecurring gate to conditions | Expr/Struct | Open — BL-24, BL-36 |
| R-05 | B | One-off re-capture reports SUCCESS falsely. `Filter_Existing_Section_By_Name` uses 43-char name match — nothing written but user told it saved. | Flow B exports | False SUCCESS | H | Filter by `pagesUrl` | Expr | Open — BL-25 |
| R-06 | B | Existing page found by date substring. Fallback creates untitled page with no mapping update. | `Filter_Pages_By_Title` | Wrong-page append | M | Match by page id | Expr | Open — BL-26 |
| R-07 | Topic → B → D | **CLOSED.** US-format EndTime is intentional. Scratch Diagnostics confirmed identical tick values. | Scratch Diagnostics 23 Sep | — | — | None | — | **Closed** |
| R-08 | Topic → B | C10 contract confirmed: `text_3` = HTML skeleton only; `text_7` = JoinUrl. `ARCHITECTURE.md` wrong. | Published YAML | Docs misleading | M | Update `ARCHITECTURE.md` | Docs | Open — BL-32 |
| R-09 | A | `/meet/` format invites get empty JoinUrl → poison row pattern. All current meetings use `/l/meetup-join/` (Q4 confirmed) so no immediate risk — blocked by FD04 guard anyway. | session-2026-09-18; Q4 | Future meetings with `/meet/` links never filled | M | Handle both formats in FA12B + FA29D–F | Expr | Open — BL-28 |
| R-10 | C | FC01 can re-select a captured row when duplicates exist. | FC01 `$filter` | Endless re-append | M | Add `and ChatCaptured eq 0` | Expr | Open — BL-29 |
| R-11 | B | Hard failures return nothing to Topic. No Scope/catch. | Tail run-afters | Generic error; duplicate pages on retry | M | Scope + catch | Struct | Open — BL-30 |
| R-12 | C | Chat capped at 50 messages. | FC07 URI | Truncation on busy meetings | L | Document; paging structural | Struct | Open — BL-31 |
| R-13 | B | UJ3b delete failure blocks whole run. | IsRecurring runAfter SUCCEEDED | One delete fail = capture fail | L | Add Failed to run-after | Settings | Open — BL-33 |
| R-14 | B | Re-capture appends full skeleton with misleading text. | `Compose_UpdateHtmlFragment` | Duplicate headings | L | Header + datestamp only | Expr | Open — BL-34 |
| R-15 | B | `outbranchresult` returns `varFinalMatchCount`. | Respond body | Diagnostics only | L | Coalesce branch results | Expr | Open — BL-11 |
| R-16 | A/B/Topic/C | Dead or literal logic (list below). | Exports | Confusion; corruption exposure | L | After stable sessions | Struct | Open — BL-14 |
| R-17 | Docs | `ARCHITECTURE.md` drift — confirmed items: V3 connector not V4; FD03 no date filter; Flow C appends to `body`; C10 contract; one-off prefix is `Evt Mtg -` not a fixed section (Q2). | Exports + session log + Q2/Q3 | Wrong design assumptions in future sessions | M | Rewrite affected sections | Docs | Open — BL-32 |

**R-16 dead or literal logic:** FA12 appends literal `"json(concat(...` to unused `varCandidates`; FA14 uses `item()` outside loop; FA15–FA26 never reached; FA28–FA30 always overwritten by C6D; Flow B D2 branch unreachable; `Compose_IgnoreSeriesMasterId` literal `''`; `outpagehtml`/`outupdatehtmlfragment` unbound; Topic C9B `PageTitle` unused; Flow C `AllowFallback` unused.

---

## 3. Fixes applied — complete record

| Fix | Status | Date | Detail |
|---|---|---|---|
| Row 2 deleted | Done | 23 Sep | Winning Together Peak call removed from SP list |
| FD04b JoinUrl guard | Done | 23 Sep | `not(empty(coalesce(item()?['JoinUrl'], '')))` |
| FD04b PageSelfUrl guard | Done | 23 Sep | `not(empty(coalesce(item()?['PageSelfUrl'], '')))` |
| FD04 JoinUrl guard | Done | 23 Sep | Same addition to recurring filter |
| FD04 PageSelfUrl guard | Done | 23 Sep | Same addition to recurring filter |
| Flow D published | Done | 23 Sep | Clean run confirmed |
| R-07 normalisation fix | Not needed | 23 Sep | Scratch Diagnostics confirmed ticks() parses correctly |

---

## 4. Candidate fix expressions (open findings)

**R-02 — FC16 run-after:** require both FC05t (recap append) and FC15 (chat) to have Succeeded before setting ChatCaptured = true. If FC05t fails, nothing was written — correct to leave uncaptured. If FC15 fails after FC05t succeeded, recap is on the page but chat is missing — leave uncaptured, accept duplicate recap on next retry.

**R-04 (F3/BL-24/BL-36) — requires Opus design session.** Need to: add IsRecurring gate to both Condition expressions; remove or gate the `Mtg -` SafeSectionName path; ensure `Evt Mtg -` is the sole one-off section prefix. Expressions in prior draft §4 are a starting point only — full Flow B logic review needed.

**R-05 (F4/BL-25):**
- `Condition_Recurring_TargetSection`: `@not(empty(first(coalesce(body('Filter_Existing_Mapping'), body('OF01_—_Filter_Existing_Mapping_OneOff'), createArray()))?['SectionPagesUrl']))` equals `true`
- `Filter_Existing_Section_By_Name` where: `@equals(item()?['pagesUrl'], variables('varTargetSectionPagesUrl'))`

**R-06 (BL-26):** `Filter_Pages_By_Title` where: `@equals(item()?['id'], outputs('Compose_ExistingPageId'))`

**R-09 (BL-28)** (Scratch Diagnostics proof first):
```
@if(contains(coalesce(item()?['body'],''),'href="https://teams.microsoft.com/l/meetup-join/'), concat('https://teams.microsoft.com/l/meetup-join/', first(split(split(coalesce(item()?['body'],''),'href="https://teams.microsoft.com/l/meetup-join/')[1],'"'))), if(contains(coalesce(item()?['body'],''),'href="https://teams.microsoft.com/meet/'), concat('https://teams.microsoft.com/meet/', first(split(split(coalesce(item()?['body'],''),'href="https://teams.microsoft.com/meet/')[1],'"'))), ''))
```

**R-10 (BL-29):** FC01 `$filter` — append ` and ChatCaptured eq 0`.

**R-14 (BL-34):** `Compose_UpdateHtmlFragment`: `@concat('<hr><p><em>Re-captured by Meeting Capture Agent on ', formatDateTime(utcNow(), 'd MMM yyyy HH:mm'), ' UTC.</em></p>')`

**R-15 (BL-11):** `outbranchresult`: `@{coalesce(outputs('Compose_Branch_Result'), outputs('Compose_Branch_Result_NoMatch'))}`

---

## 5. User journeys (final)

| Journey | Recurring | One-off |
|---|---|---|
| New capture | OK | Creates both `Evt Mtg -` and `Mtg -` sections (BL-36) |
| Re-capture, same occurrence | Second skeleton appended (BL-34); may pick wrong page (BL-26) | False SUCCESS for existing mapping (BL-25) |
| No-match day + navigation | OK | OK |
| Multi-match selection | OK | OK |
| Single match | Prompt to type "1" — acceptable (Q8) | Same |
| Auto fill, recap available | Stable; duplicate recap if chat fails (BL-22) | Working for valid JoinUrl |
| Auto fill, no recap | Chat only — correct | Same |
| No Teams link / empty JoinUrl | Blocked by FD04 guard | Same |
| `/meet/` format invite | Blocked by guard; no fill (BL-28) — no current meetings affected (Q4) | Same |
| Future meeting (row 1, 2 Oct) | Fill will fire at EndTime + 30 min; timing confirmed correct | — |

---

## 6. Recommended work order (final)

1. **Session 1 — Flow C quick wins (Sonnet).**
   - Apply R-02 fix (FC16 run-after corrected per Q6 decision).
   - Apply R-10 (FC01 ChatCaptured guard).
   - Reset one test row; confirm clean end-to-end fill including recap and chat.

2. **Session 2 — Flow B one-off correctness (Opus to design, Sonnet to edit).**
   - BL-36 (duplicate `Evt Mtg -` / `Mtg -` sections) — requires full Flow B logic review.
   - BL-24 (IsRecurring gate on conditions).
   - BL-25 (false SUCCESS on re-capture — filter by pagesUrl).
   - BL-26 (page match by id).
   - BL-07, BL-34, BL-11.
   - Test matrix: recurring new, recurring existing, one-off new, one-off existing, one-off re-capture.

3. **Session 3 — docs (Sonnet).**
   - BL-32 (rewrite `ARCHITECTURE.md` — all confirmed drift now documented above).
   - BL-17 (known-good values merge; add C and D references).

4. **Session 4 — R-09 (Sonnet, after any `/meet/` link is found in the field).**
   - Scratch Diagnostics proof, then FA12B + FA29D–F expression update.

5. **Later — structural (Opus to design).**
   - BL-08, BL-30, BL-10, BL-14.

---
*v1 created 22 Sep 2026. v2: SP list data. v3: Topic YAML + Q3 + session log + R-18 closed + FD04/FD04b fixes. v4: R-07 closed (Scratch Diagnostics). v5 (final): Q2/Q4/Q6/Q8 answered; R-02 fix corrected; BL-36 added; review complete.*
