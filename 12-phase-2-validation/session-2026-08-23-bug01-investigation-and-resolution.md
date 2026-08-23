# Session note — 23 August 2026 (BUG-01 investigation and resolution)

**Context:** Continuation from 22 Aug evening session. Goal per handover: investigate BUG-01 (second occurrence of recurring series overwriting first page) from Activity trace evidence before building anything, following the evidence-first working method.

**Model/effort:** Sonnet 4.6, Standard for most of the session; the corruption diagnosis and root-cause chain-tracing stayed within Standard throughout as the evidence made each step fairly clear-cut.

---

## Part 1 — Pre-session Flow Checker sweep

Per standing instruction, ran Flow Checker on Flow A, Flow B, and the Topic before touching anything.

**Flow A: 2 errors found** — first confirmed corruption incident in Flow A (previously only Flow B and Email Triage had been affected). Two `SetVariable` actions had blanked `value` fields:
- `FA33A_Set_varCandidateListText_Empty` — correct value `@string('')` (an expression, not a literal `""` — the literal form found during corruption was itself a degraded/incorrect state)
- `FA34A_Set_varCandidateIndex_One` — correct value `1`

Both restored, confirmed via Peek Code, Flow Checker 0 errors, published. **New reference doc created:** `known-good-values-flow-a-reference.md`, seeded from a full Peek Code capture, since Flow A previously had no dedicated known-good-values doc (only Flow B did).

Flow B and Topic: both came back clean on the initial sweep.

---

## Part 2 — First capture attempt reveals a data/write problem (not yet BUG-01)

David ran a first live capture on the "SC&L FLT Stand-up" series and hit a `BadRequest` on `Create_Mapping_Item_Recurring`:
> "The list item could not be added or updated because duplicate values were found in the following field(s) in the list: [SeriesMasterId]"

Trace showed: `Get_items` found an existing row (ID 279) for this series, created 41 seconds earlier by an *even earlier* run in the same session. That row was a **skeleton/orphaned row** — `Title`/`SeriesMasterId`/`MeetingTitle`/`Status`/`OccurrenceDate` populated, but `SectionPagesUrl` and other Section/Page fields blank, indicating an earlier run had created the mapping row but failed before completing OneNote section/page creation.

This surfaced two separate, real issues:
1. A live example of the **stale/orphaned mapping row** scenario UJ3b is designed to address (not yet built).
2. `Create_OneNote_Page` failing separately with "the section id given in the input is invalid" once the flow reached that far, because `varTargetSectionPagesUrl` was being pulled from the orphaned row's blank `SectionPagesUrl`.

Deferred fixing this (data issue, not flow logic) and proceeded to the actual BUG-01 investigation with David supplying full Peek Code for the relevant branches.

---

## Part 3 — BUG-01 root cause: `varFinal*` variables silently corrupted

Full Peek Code review of `Condition_Mapping_Exists`, `Condition_Should_Write_Mapping`, and surrounding actions revealed:

`varFinalExistingPageSelfUrl_1`, `varFinalPageDecision_1`, `varFinalMatchCount_1` — the three `SetVariable` actions responsible for carrying `Compose_ExistingPageSelfUrl`, `Compose_PageDecision`, and `Compose_Match_Count`'s outputs into the variables actually used downstream — were **all blanked** (`value: ""`), despite Flow Checker not flagging any error on them (a known Flow Checker blind spot, consistent with `bug-2026-08-16-of05b-silent-empty-value-flowchecker-blind.md`).

With `varFinalMatchCount` empty, `Condition_Mapping_Exists`'s guard (`@greater(int(if(empty(variables('varFinalMatchCount')), '0', variables('varFinalMatchCount'))), 0)`) always evaluated to `0 > 0` = false, regardless of what `Filter_Existing_Mapping` actually found. This meant the flow **always** routed to the `CREATE_REQUIRED`/`Condition_Should_Write_Mapping` branch, even when a mapping row for that series+occurrence already existed — explaining both today's `BadRequest` duplicate-insert crash and, by the same mechanism, the originally reported BUG-01 second-occurrence overwrite behaviour.

**Fix:** restored all three actions to their documented known-good values (matching `known-good-values-master-reference.md`, which had these correct all along — confirming this was corruption, not a design defect):
- `varFinalExistingPageSelfUrl_1` → `@outputs('Compose_ExistingPageSelfUrl')`
- `varFinalPageDecision_1` → `@outputs('Compose_PageDecision')`
- `varFinalMatchCount_1` → `@string(outputs('Compose_Match_Count'))`

---

## Part 4 — 21-action corruption incident mid-session

Immediately after the Part 3 fix, re-opening the Designer surfaced **21 fresh operation errors** — a large-scale corruption hit distinct from the 3-action one just fixed. Recovered systematically against `known-good-values-master-reference.md`, action by action, in Flow Checker's listed order. All 21 restored and confirmed.

**A new issue surfaced during this recovery:** pasting the seven-value `Set_varOutStatus` expression from the reference doc produced a `TemplateValidationError` — `expected token 'EndOfData' and actual 'RightParenthesis'`. Paren-balance check (46 open vs 47 close) confirmed the reference doc itself had a transcription error — one extra trailing `)`. Corrected, re-verified balanced (46/46), pasted successfully, Flow Checker 0 errors, published. **The master reference doc has been corrected** so this typo isn't propagated in future recovery incidents.

---

## Part 5 — Retest reveals the real structural root cause

With the corruption and paren-typo fixed, retested the original capture: `Create_Mapping_Item_Recurring` failed again with the same duplicate-`SeriesMasterId` error — but this time `Filter_Existing_Mapping` and `Compose_Match_Count` were confirmed correct (0 matches for the new `OccurrenceDate`), meaning the flow *correctly* attempted a fresh insert. The insert itself was being rejected by SharePoint.

Checked the `RecurringMeetingSectionMap` list's `SeriesMasterId` column settings: **"Enforce unique values" = Yes.** This SharePoint list-level constraint meant *any* second row for the same `SeriesMasterId` would always be rejected, regardless of `OccurrenceDate` — independent of and unfixable via flow logic. The flow's own `Filter_Existing_Mapping` where-clause already correctly enforces "one row per series+occurrence" at the logic layer, making the column-level constraint both redundant and actively harmful.

**Fix:** set "Enforce unique values" → No on the `SeriesMasterId` column.

---

## Part 6 — Full validation

Cleared the `RecurringMeetingSectionMap` list and the test OneNote section/pages for a clean baseline. Ran two sequential captures on the same recurring series:

- **Occurrence 1 (24 Aug):** 1 SharePoint row created, fully populated. 1 OneNote page created ("SCandL FLT Stand-up - 24 Aug 2026").
- **Occurrence 2 (31 Aug):** 2nd SharePoint row created successfully (distinct `OccurrenceDate`, no collision). 2nd OneNote page created ("SCandL FLT Stand-up - 31 Aug 2026"), fully separate from the first.

Confirmed via SharePoint list view (both rows visible, correct `OccurrenceDate` values) and OneNote page inspection (both pages present, both intact, neither overwritten).

**BUG-01 is resolved.**

---

## Status at end of session

| Item | Status |
|---|---|
| Flow A corruption (2 actions) | ✅ Fixed, published |
| Flow A known-good-values reference | ✅ Created (`known-good-values-flow-a-reference.md`) |
| Flow B `varFinal*` corruption (3 actions, BUG-01 primary cause) | ✅ Fixed, published |
| Flow B 21-action corruption incident | ✅ Fixed, published |
| `Set_varOutStatus` paren-balance typo in master reference | ✅ Corrected in doc |
| BUG-01 (second occurrence overwrite/collision) | ✅ **RESOLVED and validated end-to-end** |
| Orphaned/skeleton mapping row (ID 279) | ✅ Manually deleted as part of cleanup |
| `SeriesMasterId` "Enforce unique values" constraint | ✅ Disabled (structural root cause fix) |
| UJ3b — automatic stale-row cleanup | Not built (residual value even post-fix, for future orphaned-row scenarios) |
| `Condition_Should_Write_Mapping` explicit match-count guard | Not built — flagged as defense-in-depth candidate, not urgent given root cause now fixed upstream |
| FR-01, FR-02, FR-03 | Not started this session |

## Recommended next session

1. **FR-02 — holiday/leave filter.** Low complexity, high practical value, queued from 22 Aug backlog.
2. **FR-01 — chronological candidate list ordering.** Confirm current Graph API ordering behaviour first.
3. **FR-03 — link shortening.** Medium complexity, needs option evaluation.
4. **UJ3b — automatic stale-row cleanup.** Lower urgency now that the unique-constraint fix removes the main failure mode it was designed to guard against, but still worth building for resilience.
5. Consider submitting the Microsoft support ticket — today's session adds a 12th+ corruption data point (Flow A hit for the first time, plus a 21-action Flow B incident), reinforcing the case already made in `microsoft-discussion-brief-corruption-bug.md`.

---
*Written 23 August 2026.*
