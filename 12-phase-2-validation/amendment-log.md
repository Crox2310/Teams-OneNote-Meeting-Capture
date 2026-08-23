# Amendment Log

Formal, numbered record of confirmed defects found and fixed in the Teams → OneNote Meeting Capture flows (Flow A, Flow B) and Topic, from 27 July 2026 onward. Each entry links back to the full investigation doc it was traced in — this log is a compact index, not a replacement for those docs.

**Numbering:** `AMEND-YYYY-MM-DD-NNN`, sequential per day.

**Scope note:** this log starts from 27 July 2026, when the pattern of numbered amendments began. A substantial amount of earlier fix history (June 2026 — Flow A/B bug-fixing campaigns, structural fixes, the v5 UX rebuild) exists in dated handover docs in this folder but was not logged under this numbering convention at the time. Rather than retroactively assign numbers to that history (risking inaccuracy, since some of those docs describe iterative in-session fixes rather than single discrete amendments), it's left as-is in its original handover docs. See "Pre-log history" at the bottom for pointers.

---

## AMEND-2026-07-27-001 — `Condition Is Genuine Existing Page` unreachable (initial fix, superseded same day)

**Flow:** Flow B
**Defect:** condition checked `variables('varOneNoteResolverResult') == 'Exists'` but no action ever sets it to `"Exists"`. Condition was unreachable.
**Fix:** changed to `contains(createArray('ExistingMapping','ExistingSection'), variables('varOneNoteResolverResult'))`.
**Full trace:** `2026-07-27-condition-is-genuine-existing-page-defect.md`.

## AMEND-2026-07-27-002 — Two `SetVariable` actions missing `value` keys entirely

**Flow:** Flow B
**Defect:** `Set varOneNoteResolverResult ExistingMapping` and `Set_varOneNoteResolverResult_Exists_OneOff` both had no `value` key. First confirmed instance of the platform corruption pattern.
**Fix:** `value: "ExistingMapping"` and `value: "ExistingSection"` respectively.
**Full trace:** `2026-07-27-condition-is-genuine-existing-page-defect.md`.

---

## AMEND-2026-08-16-001 — `Get_Pages_In_Section_Existing_Branch` sectionId format mismatch

**Flow:** Flow B
**Defect:** `sectionId` sourced from bare section GUID; `GetPagesInSection` requires `pagesUrl`-style value.
**Fix:** changed to `@items('Apply_to_each_Existing_Section')?['pagesUrl']`.
**Full trace:** `handover-2026-08-16-morning-sectionid-fix-and-page-title-gap.md`.

## AMEND-2026-08-16-002 — Bug 9: `NotFound` on `Update_page_content_Existing_Branch` — temporary workaround

**Flow:** Flow B
**Defect:** pages never given real titles, defaulting to "Untitled Page". Title-based matching always returned zero results.
**Fix (temporary):** `Compose_RealExistingPageId` changed to take first page directly from section. Superseded by FB-04.
**Full trace:** `handover-2026-08-16-bug9-closed-workaround-confirmed.md`.

## AMEND-2026-08-16-003 — Permanent page-title fix, recurring branch

**Flow:** Flow B
**Fix:** post-creation title-set via `UpdatePageContent` connector using live-verified page ID.
**Full trace:** `handover-2026-08-16-page-title-fix-recurring-confirmed.md`.

## AMEND-2026-08-16-004 — Permanent page-title fix, one-off branch — built, not yet tested

**Flow:** Flow B
**Fix:** identical five-action pattern for `Create_Page_OneOff`.
**Full trace:** `handover-2026-08-16-oneoff-title-fix-built-unconfirmed.md`.

---

## AMEND-2026-08-20-001 — Issue #3: Date entry format sensitivity

**Flow:** Topic
**Defect:** only `d MMM` format accepted. Slash-format dates not recognised.
**Fix:** `C6C_Check_Date` extended with slash-format regex match and parser. `C6D_Check_Number` added to prevent plain number selection being intercepted.
**Live-verified:** confirmed.
**Full trace:** `session-2026-08-20-issue3-datehandling.md`.

## AMEND-2026-08-21-001 — Issue #2: Recapture discards updated content

**Flow:** Flow B
**Defect:** `Compose_UpdateHtmlFragment` discarded organiser's updated body content on recapture.
**Fix:** changed to `concat(notice, triggerBody()?['text_3'])`.
**Live-verified:** confirmed.
**Full trace:** `session-2026-08-21-fb-progress-and-incidents.md`.

## AMEND-2026-08-21-002 — Issue #1 foundation: OccurrenceDate flowing end-to-end (FB-01/02/03)

**Flow:** Flow B + Topic
**Fix:** `Filter_Existing_Mapping` updated to match on `SeriesMasterId` AND `OccurrenceDate`. `text_5` added to trigger schema. Mapping row write updated to include `OccurrenceDate`. `C9B_Set_PageTitle` updated to produce dated title.
**Full trace:** `session-2026-08-21-fb-progress-and-incidents.md`.

---

## AMEND-2026-08-22-001 — Issue #1 completion: Date-based page matching (FB-04/05)

**Flow:** Flow B
**Fix:** `Filter_Pages_By_Title` updated to match on date string. `Compose_RealExistingPageId` updated to consume filter output. `Compose_SafePageTitle`/`_OneOff` updated to append dated title.
**Live-verified:** full create → recapture cycle confirmed. Multiple series, multiple dates.
**Full trace:** `session-2026-08-22-backlog-reduction-and-fb04-confirmed.md`.

## AMEND-2026-08-22-002 — OutStatus differentiated to 6 values

**Flow:** Flow B + Topic
**Fix:** four new Compose actions added. `Set_varOutStatus` replaced with six-value expression. `C11_Check_OutStatus` updated to check `"SUCCESS"`.
**Live-verified:** confirmed.
**Full trace:** `session-2026-08-22-outstatus-differentiation.md`.

## AMEND-2026-08-22-003 — BadGateway: native Create item connector

**Flow:** Flow B
**Fix:** raw HTTP POST replaced with native SharePoint `Create item` on both paths. Status code check updated to 201.
**Live-verified:** confirmed.
**Full trace:** `session-2026-08-22-fa16-and-badgateway-fix.md`, `session-2026-08-22-badgateway-verification.md`.

## AMEND-2026-08-22-004 — FA43 coalescing gap

**Flow:** Flow A
**Fix:** `isrecurring` and `seriesmasterid` in `FA43_Respond_to_agent` updated to coalesce across all three paths.
**Full trace:** `session-2026-08-22-fa43-and-endofday.md`.

## AMEND-2026-08-22-005 — Section name sanitiser character gap

**Flow:** Flow B (3 actions)
**Fix:** five additional `replace()` pairs added: `|`, `#`, `'`, `%`, `~`.
**Full trace:** `session-2026-08-22-backlog-reduction-and-fb04-confirmed.md`.

## AMEND-2026-08-22-006 — Link-format bug

**Flow:** Flow B
**Fix:** `Set_varOutputPageLink_Existing` updated to `@first(body('Filter_Existing_Mapping'))?['PageWebUrl']`.
**Full trace:** `session-2026-08-22-backlog-reduction-and-fb04-confirmed.md`.

## AMEND-2026-08-22-007 — FA16 defensive guard

**Flow:** Flow A
**Fix:** round-trip guard added to `FA16_Compose_SelectedIndex`.
**Live-verified:** confirmed.
**Full trace:** `session-2026-08-22-fa16-and-badgateway-fix.md`.

## AMEND-2026-08-22-008 — UJ5b: Explicit Cancel at selection prompt

**Flow:** Topic
**Fix:** `C6E_Check_Cancel` condition added to `conditionGroup_BsGPk1`. Prompt text updated to mention `- C to cancel`. Unrecognised-input message updated.
**Full trace:** `session-2026-08-22-evening-uj345.md`.

## AMEND-2026-08-22-009 — UJ5a: Retry option on error

**Flow:** Topic
**Fix:** `C12_Error` elseActions updated to offer retry (`R` loops back to `invokeFlowAction_bWHHeg`). Clean exit message on anything else.
**Full trace:** `session-2026-08-22-evening-uj345.md`.

## AMEND-2026-08-22-010 — UJ4b: Blank SeriesMasterId guard

**Flow:** Flow B
**Fix:** `Filter_Existing_Mapping` where clause updated to add `not(empty(triggerBody()?['text_2']))` as first condition. Prevents empty SeriesMasterId matching wrong rows.
**Full trace:** `session-2026-08-22-evening-uj345.md`.

## AMEND-2026-08-22-011 — UJ3: Stale-row detection (STALE_MAPPING)

**Flow:** Flow B + Topic
**Fix:** `STALE_MAPPING` added as seventh OutStatus value — fires when `varPageAction` is blank AND `varOneNoteResolverResult` is `ExistingMapping` or `ExistingSection`. Topic updated with specific actionable message for this case.
**Full trace:** `session-2026-08-22-evening-uj345.md`.

## AMEND-2026-08-22-012 — UJ4a: Section disambiguation (count > 1 blocked)

**Flow:** Flow B
**Fix:** `Condition_Section_Exists_Recurring` expression changed from `greater(count, 0)` to `equals(count, 1)`. New nested `Condition_Section_Count_Is_Zero` added in the else branch: true (count=0) creates section, false (count>1) does nothing — flow continues with blank section URL, `OutStatus` evaluates to `SETUP_SECTION_AMBIGUOUS`.
**Full trace:** `session-2026-08-22-evening-uj345.md`.

---

## AMEND-2026-08-23-001 — Flow A: `FA33A`/`FA34A` corruption (first-ever Flow A incident)

**Flow:** Flow A
**Defect:** `FA33A_Set_varCandidateListText_Empty` and `FA34A_Set_varCandidateIndex_One` both had blanked `value` fields — first confirmed corruption incident on Flow A (previously only Flow B and Email Triage had been affected).
**Fix:** `FA33A` → `@string('')`, `FA34A` → `1`.
**Live-verified:** confirmed, Flow Checker clean, published.
**Full trace:** `session-2026-08-23-bug01-investigation-and-resolution.md`.

## AMEND-2026-08-23-002 — BUG-01: `varFinal*` corruption causing second-occurrence overwrite

**Flow:** Flow B
**Defect:** `varFinalExistingPageSelfUrl_1`, `varFinalPageDecision_1`, `varFinalMatchCount_1` all had blanked `value` fields. With `varFinalMatchCount` always empty, `Condition_Mapping_Exists`'s guard always evaluated false regardless of what `Filter_Existing_Mapping` actually found — routing every capture to the CREATE_REQUIRED branch even when a mapping already existed. Root cause of the originally reported BUG-01 (second occurrence of a recurring series overwriting the first).
**Fix:** restored all three to documented known-good values (`@outputs('Compose_ExistingPageSelfUrl')`, `@outputs('Compose_PageDecision')`, `@string(outputs('Compose_Match_Count'))`).
**Live-verified:** confirmed end-to-end with two sequential occurrence captures, each producing its own SharePoint row and OneNote page.
**Full trace:** `session-2026-08-23-bug01-investigation-and-resolution.md`.

## AMEND-2026-08-23-003 — Flow B: 21-action corruption incident

**Flow:** Flow B
**Defect:** 21 actions found with blanked `value` fields at session start, surfaced immediately after AMEND-002's fix.
**Fix:** all 21 restored from `known-good-values-master-reference.md`, in Flow Checker's listed order.
**Full trace:** `session-2026-08-23-bug01-investigation-and-resolution.md`.

## AMEND-2026-08-23-004 — `Set_varOutStatus` paren-balance typo in reference doc

**Flow:** Flow B (and the reference doc itself)
**Defect:** the seven-value `Set_varOutStatus` expression as recorded in `known-good-values-master-reference.md` had one extra trailing closing parenthesis (47 close vs 46 open), causing a `TemplateValidationError` when pasted verbatim during AMEND-003's recovery.
**Fix:** corrected expression (46/46 balanced) applied to the flow and to the reference doc, with a correction note added to prevent the error recurring from an old copy.
**Full trace:** `session-2026-08-23-bug01-investigation-and-resolution.md`.

## AMEND-2026-08-23-005 — Root cause: `SeriesMasterId` SharePoint unique-constraint

**List:** `RecurringMeetingSectionMap` (SharePoint)
**Defect:** the `SeriesMasterId` column had "Enforce unique values" set to Yes — a list-level constraint that blocked any second mapping row for the same recurring series regardless of `OccurrenceDate`, independent of and unfixable via flow logic. This was the true structural root cause underlying BUG-01, on top of AMEND-002's corruption fix.
**Fix:** "Enforce unique values" set to No. Uniqueness for "one row per occurrence" remains correctly enforced at the logic layer by `Filter_Existing_Mapping`'s combined `SeriesMasterId` + `OccurrenceDate` match.
**Live-verified:** confirmed via a clean two-occurrence capture test producing two separate, correctly populated rows.
**Full trace:** `session-2026-08-23-bug01-investigation-and-resolution.md`.

## AMEND-2026-08-23-006 — FR-03: OneNote link shortening via hyperlink

**Flow:** Topic
**Fix:** `C12_Success` message changed from displaying the raw `oneNoteWebUrl` (~250+ chars) to a markdown hyperlink `[Open in OneNote]({Topic.OutCreatedPageLink})`. Investigated and ruled out swapping the underlying URL field first — live evidence showed none of Microsoft's three OneNote API URL variants (`oneNoteWebUrl`, `oneNoteClientUrl`, `oneNoteEmbedUrl`) were meaningfully shorter, and `oneNoteClientUrl` uses a non-`https://` scheme with reliability risk.
**Live-verified:** confirmed, Teams renders a clickable "Open in OneNote" link.
**Full trace:** `session-2026-08-23-part2-fr03-fr02-bug02.md`.

## AMEND-2026-08-23-007 — FR-02: Holiday/leave/period/admin-block candidate list filter

**Flow:** Flow A
**Fix:** new Filter array action `FA09B_Filter_ExcludeLeaveAndPeriodEntries` inserted after `FA09_RAW_CandidateArray_DoNotUseDownstream`, excluding 11 patterns (holiday, leave, A/L, on leave, OOO/out of office, bank holiday, Smarter Working, `P<n> W<n> (Week <n>)` period reminders, Manage Email & Teams, Quiet Hour). Six downstream consumers (`FA11`, `FA13`, `FA28`, `FA19`, `FA35`) repointed from `FA09` to `FA09B`, with `FA09` itself left untouched to protect existing wiring.
**Bugs found and fixed during build (see full trace for detail):** a regex over-escaping issue caused by Designer paste round-tripping; `isMatch()` is not a valid WDL function (Power Fx only) — rebuilt as a compound `startsWith`/`contains` check; a field-swap slip during the six-action repoint (`FA11` and `FA13`'s intended expressions were initially swapped).
**Live-verified:** confirmed on a real multi-entry day — all filtered patterns correctly excluded, genuine meetings correctly retained and numbered.
**Full trace:** `session-2026-08-23-part2-fr03-fr02-bug02.md`.

## AMEND-2026-08-23-008 — BUG-02: Zero-match day had no P/N/date navigation

**Flow:** Topic
**Defect:** `C4_Check_MatchCount`'s true (zero-match) branch only sent a message instructing the user to type P/N/date — no `Question` node captured the reply, so typing "N" on a zero-match day fell through to generic intent-recognition failure. Pre-existing gap, surfaced for the first time by FR-02 creating the first genuinely zero-match test day.
**Fix:** added `question_C4B_AskNav` and `conditionGroup_C4C_Nav`, mirroring the proven P/N/date/Cancel pattern from the has-matches branch, routing back to `C2_Call_FlowA_Initial`.
**Live-verified:** confirmed — "N" on a zero-match day now correctly re-searches the next day.
**Full trace:** `session-2026-08-23-part2-fr03-fr02-bug02.md`.

## AMEND-2026-08-23-009 — Personal admin-block patterns added to FR-02 filter

**Flow:** Flow A
**Fix:** `FA09B`'s where-clause extended with two further patterns: `manage email & teams`, `quiet hour` — same treatment as the original 9 FR-02 patterns.
**Live-verified:** confirmed.
**Full trace:** `session-2026-08-23-part2-fr03-fr02-bug02.md`.

## AMEND-2026-08-23-010 — FR-01: Chronological candidate list ordering

**Flow:** Flow A
**Defect:** confirmed via live Activity trace that Microsoft Graph does not return calendar events in chronological order — verified against real evidence rather than assumed.
**Fix:** new Compose action `FA09C_Sort_CandidatesByStartTime` inserted after `FA09B`, using `@sort(body('FA09B_Filter_ExcludeLeaveAndPeriodEntries'), 'start')`. Same six downstream consumers repointed from `FA09B` to `FA09C`.
**Bug found and fixed safely via `PA - Scratch Diagnostics` before touching production:** WDL's `sort()` does not accept lambda/arrow-function syntax (`(item) => ...`); correct signature is `sort(array, 'propertyName')` with the key as a plain string, confirmed via the scratch flow's runtime error message before being applied live.
**Live-verified:** confirmed — candidate list now displays in correct chronological order.
**Full trace:** `session-2026-08-23-part3-fr01.md`.

---

## Pre-log history (not numbered, see original docs)

Significant fixes made before this log began, documented in their own dated handover docs:

- **P/N navigation root causes** — `2026-07-16-flow-a-pn-navigation-root-cause.md`, `2026-07-18-pn-navigation-topic-fixes.md`
- **Date-jump feature build and UJ1–UJ5 validation** — `2026-07-20-date-jump-feature-and-uj-validation.md`
- **June 2026 Flow A/B bug-fixing campaign** — `handover-2026-06-*.md` series and `STRUCTURAL-FIXES-2026-06-21.md`
- **v5 UX rebuild** — `design-v5-ux-rebuild-2026-06-27.md`
