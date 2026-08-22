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

## Pre-log history (not numbered, see original docs)

Significant fixes made before this log began, documented in their own dated handover docs:

- **P/N navigation root causes** — `2026-07-16-flow-a-pn-navigation-root-cause.md`, `2026-07-18-pn-navigation-topic-fixes.md`
- **Date-jump feature build and UJ1–UJ5 validation** — `2026-07-20-date-jump-feature-and-uj-validation.md`
- **June 2026 Flow A/B bug-fixing campaign** — `handover-2026-06-*.md` series and `STRUCTURAL-FIXES-2026-06-21.md`
- **v5 UX rebuild** — `design-v5-ux-rebuild-2026-06-27.md`
