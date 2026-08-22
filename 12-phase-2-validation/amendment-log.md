# Amendment Log

Formal, numbered record of confirmed defects found and fixed in the Teams → OneNote Meeting Capture flows (Flow A, Flow B) and Topic, from 27 July 2026 onward. Each entry links back to the full investigation doc it was traced in — this log is a compact index, not a replacement for those docs.

**Numbering:** `AMEND-YYYY-MM-DD-NNN`, sequential per day.

**Scope note:** this log starts from 27 July 2026, when the pattern of numbered amendments began. A substantial amount of earlier fix history (June 2026 — Flow A/B bug-fixing campaigns, structural fixes, the v5 UX rebuild) exists in dated handover docs in this folder but was not logged under this numbering convention at the time. Rather than retroactively assign numbers to that history (risking inaccuracy, since some of those docs describe iterative in-session fixes rather than single discrete amendments), it's left as-is in its original handover docs. See "Pre-log history" at the bottom for pointers.

---

## AMEND-2026-07-27-001 — `Condition Is Genuine Existing Page` unreachable (initial fix, superseded same day)

**Flow:** Flow B (`PA - Resolve OneNote Meeting Section`)
**Defect:** `Condition Is Genuine Existing Page` checked `variables('varOneNoteResolverResult') == 'Exists'`, but no action anywhere in the flow ever sets that variable to the literal string `"Exists"` — the real values in use are `"ExistingMapping"`, `"ExistingSection"`, `"CreatedSection"`. The condition was structurally unreachable.
**Real-world impact confirmed live:** recapturing a recurring meeting ("HoP - Focus Time") that already had a page produced a **second, duplicate page** for the same date instead of updating the existing one — confirmed directly in OneNote.
**Fix applied:** changed the condition's expression to `contains(createArray('ExistingMapping','ExistingSection'), variables('varOneNoteResolverResult'))`.
**Result:** insufficient on its own — see AMEND-2026-07-27-002. Two of the four `varOneNoteResolverResult`-setting actions were separately found to never populate a value at all, so the corrected condition still had nothing to match against on those paths.
**Full trace:** `2026-07-27-condition-is-genuine-existing-page-defect.md`, Sections 1–6.

## AMEND-2026-07-27-002 — Two `SetVariable` actions missing `value` keys entirely (Pattern 6)

**Flow:** Flow B
**Defect:** `Set varOneNoteResolverResult ExistingMapping` and `Set_varOneNoteResolverResult_Exists_OneOff` both had `SetVariable` action definitions with no `value` key present at all — the actions set the variable's *name* but supplied no expression to assign to it. This is the first confirmed instance of what's since become known as **Pattern 6** in the living audit: `SetVariable` actions silently missing their `value` key, distinct from having an empty or incorrect value.
**Fix applied:**
- `Set varOneNoteResolverResult ExistingMapping` → `"value": "ExistingMapping"`
- `Set_varOneNoteResolverResult_Exists_OneOff` → `"value": "ExistingSection"` (found already corrected by the time of the fix session — likely set in an earlier pass; cause not fully established)
**Live-verified:** recapturing the same "HoP - Focus Time" occurrence a fourth time showed `Condition Is Genuine Existing Page` evaluating True for the first time in the investigation, the full existing-page-update chain executing (`Get Sections Existing Branch` → ... → `Set varOutputPageLink Existing`), and — confirmed directly in OneNote — no new duplicate page created.
**Outstanding, not part of this fix:** cleanup of the duplicate pages and one stray "Untitled Page" created during the investigation's earlier live tests. Low risk, no flow changes required — still not done as of 31 July.
**Full trace:** `2026-07-27-condition-is-genuine-existing-page-defect.md`, Sections 7–8.

---

## Pattern 6 — confirmed recurrences after 27 July

Pattern 6 (`SetVariable` actions missing their `value` key entirely) has recurred at least twice more since the two instances fixed above:

### Confirmed, not yet fixed as of 31 July — `Set varOutputPageSelfUrl Existing`

**Flow:** Flow B
**Found:** 30 July 2026, live-traced as the proximate cause of a `BadRequest` on `Update_page_content_Existing_Branch` when recapturing a one-off meeting.
**Status:** root cause confirmed via Peek Code and live run output (30 July). Investigation subsequently found this to be a symptom of a larger gap — one-off meetings have no mapping-table memory at all, and `Condition_Should_Create_Page` itself never receives a valid decision input for one-off meetings — rather than a fix in isolation. **Full design for the complete fix (including this `value` correction as step 8 of 9) finalised 31 July.** Not yet built.
**Full trace:** `handover-2026-07-30-oneoff-badrequest-live-trace-confirmation.md` (investigation), `2026-07-31-oneoff-design-evidence-meetingid-column.md` (final design, 9-step build plan, **start here for the build**).

### Related — non-Pattern-6 but same investigation, confirmed 31 July

`Condition_Should_Create_Page` reads `outputs('Compose_PageDecision')` directly, but that Compose action only exists on the recurring path — so for every one-off meeting, this condition evaluates against a never-executed action's output. Not a missing-`value` bug like Pattern 6, but the same broader category: an expression silently referencing something that isn't there for every meeting type. Design decision made (Option A — read from a shared variable instead); see `2026-07-31-oneoff-design-evidence-meetingid-column.md`.

---

## AMEND-2026-08-16-001 — `Get_Pages_In_Section_Existing_Branch` `sectionId` format mismatch

**Flow:** Flow B, existing-page-update branch (Bug 9 investigation)
**Defect:** newly-built `Get_Pages_In_Section_Existing_Branch` action (part of the Option 1 page-lookup redesign) sourced its `sectionId` parameter from `items('Apply_to_each_Existing_Section')?['id']` — the bare section GUID. `GetPagesInSection` requires the full `pagesUrl`-style value, not a bare ID, consistent with picklist-style connector fields in this environment.
**Symptom:** `400 BadRequest — "The section id given in the input is invalid."`
**Fix applied:** changed `sectionId` to `@items('Apply_to_each_Existing_Section')?['pagesUrl']`, copying the exact expression already proven working on the pre-existing `Update_page_content_Existing_Branch` action in the same loop.
**Live-verified:** confirmed via isolated re-test; action succeeded post-fix.
**Full trace:** `handover-2026-08-16-morning-sectionid-fix-and-page-title-gap.md`.

## AMEND-2026-08-16-002 — Bug 9 (`NotFound` on `Update_page_content_Existing_Branch`) — CLOSED via temporary workaround

**Flow:** Flow B, existing-page-update branch
**Defect:** long-running Bug 9 (first logged 15 August) — pages were never given a real title at creation, defaulting to OneNote's `"Untitled Page"`. Title-based page matching (`Filter_Pages_By_Title`) therefore always returned zero results, and the resulting empty `pageId` caused `Update_page_content_Existing_Branch` to fail.
**Fix applied (temporary):** `Compose_RealExistingPageId` changed to take the section's first page directly from `Get_Pages_In_Section_Existing_Branch`'s live output, bypassing the broken title match. Explicitly a stopgap — valid only while sections hold exactly one page; will break once real multi-page sections are recaptured.
**Live-verified:** run trace all-green, raw `204` response from `UpdatePageContent`, and direct visual confirmation of the appended "Automated update" block on the real OneNote page.
**Superseded by:** AMEND-2026-08-16-003 for pages created after that fix lands; existing untitled pages still rely on this workaround until a one-time backfill or natural rollover.
**Full trace:** `handover-2026-08-16-bug9-closed-workaround-confirmed.md`.

## AMEND-2026-08-16-003 — Permanent page-title fix, recurring branch (`Create_OneNote_Page`)

**Flow:** Flow B, recurring page-creation path
**Defect:** root cause of AMEND-2026-08-16-002 — `Create_OneNote_Page`'s `pageContent` connector field only ever accepts a body-content fragment (confirmed via Parameters tab: no dedicated `title` parameter exists on `CreatePageInSection`), so pages were never given a real title.
**Fix applied:** post-creation title-set via the `UpdatePageContent` connector's `target: "title"` update type, using a live-verified page ID rather than trusting `Create_OneNote_Page`'s own (occasionally not-yet-propagated) output — `Compose_SafePageTitle` → `Create_OneNote_Page` → `Compose_PageSelfUrl_Created` → `Get_Pages_In_Section_Recurring_PostCreate` → `Filter_Pages_By_SelfUrl_Recurring` → `Compose_ConfirmedCreatedPageId` → `Set_PageTitle_Recurring`.
**Live-verified:** run trace all-green including `Set_PageTitle_Recurring`; real OneNote page confirmed showing the correct title (not "Untitled Page") in the notebook list.
**Known follow-up issue, NOT yet resolved:** `Set_PageTitle_Recurring` intermittently fails with `404`/OneNote error 20102 even against a freshly-verified page ID — an occasional propagation-delay race narrower than the original bug but not eliminated. A Delay-based mitigation was attempted and abandoned same day due to an Express-mode platform conflict (see `handover-2026-08-16-session-close-express-mode-unstable.md`); a retry/poll (`Do until`) approach is recommended instead for a future session.
**Full trace:** `handover-2026-08-16-page-title-fix-recurring-confirmed.md`.

## AMEND-2026-08-16-004 — Permanent page-title fix, one-off branch (`Create_Page_OneOff`) — BUILT, NOT YET TESTED

**Flow:** Flow B, one-off stale-mapping edge-case path
**Defect:** same as AMEND-2026-08-16-003, mirrored for `Create_Page_OneOff`.
**Fix applied:** identical five-action pattern (`Compose_SafePageTitle_OneOff` → `Create_Page_OneOff` → `Get_Pages_In_Section_OneOff_PostCreate` → `Filter_Pages_By_SelfUrl_OneOff` → `Compose_ConfirmedCreatedPageId_OneOff` → `Set_PageTitle_OneOff`).
**Status:** Flow Checker clean, published — **but this code path was confirmed to be far narrower than assumed** (only reached via a stale-mapping scenario, not ordinary first-time one-off captures, which already route through the recurring-branch fix above) and has not yet been exercised by any test. Do not treat as confirmed working.
**Full trace:** `handover-2026-08-16-oneoff-title-fix-built-unconfirmed.md`.

---

## AMEND-2026-08-20-001 — Issue #3: Date entry format sensitivity

**Flow:** Topic (`C6C_Check_Date`)
**Defect:** date entry only accepted `d MMM` format (e.g. `23 Oct`). Slash-format dates (`dd/MM/yy`, `dd/MM/yyyy`) and other common formats were not recognised, causing the agent to fall through to the unrecognised-input handler rather than navigating to the entered date.
**Fix applied:** `C6C_Check_Date` condition extended to also match slash-format dates via `IsMatch(Topic.TopicSelectedNumber, "^\d{1,2}/\d{1,2}/\d{2,4}$")`; `C6C_Check_Date` action updated with a full slash-aware date parser using `Date()` with `Split()` to extract day/month/year components, handling both 2-digit and 4-digit years.
**Regression caught and fixed same session:** the new `else` branch intercepted plain number selection (e.g. typing `3` to select meeting 3), routing it to the unrecognised-input handler instead of the selection path. Fixed by adding explicit `C6D_Check_Number` condition (`=!IsError(Value(Topic.TopicSelectedNumber))`) before the else, with a no-op `SetVariable` to satisfy the condition requirement.
**Live-verified:** confirmed via live test. Plain number selection confirmed still working (regression check passed).
**Full trace:** `session-2026-08-20-issue3-datehandling.md`, `fix-2026-08-20-3-datehandling-resolved.md`.

## AMEND-2026-08-21-001 — Issue #2: Recapture discards organiser's updated content

**Flow:** Flow B, `Compose_UpdateHtmlFragment`
**Defect:** on recapture (existing page found), the update fragment that gets appended to the page only included the static "Automated update" header block and discarded the organiser's updated meeting body content (`text_3`). Human-entered notes in the page were preserved (correct), but the refreshed meeting details from the organiser were lost.
**Fix applied:** `Compose_UpdateHtmlFragment` inputs changed from a static string to `concat(notice, triggerBody()?['text_3'])`, appending the full current meeting body after the static header block.
**Live-verified:** confirmed via live test — recapture now appends both the header and the refreshed meeting content while preserving existing human notes.
**Full trace:** `session-2026-08-21-fb-progress-and-incidents.md`.

## AMEND-2026-08-21-002 — Issue #1 foundation: OccurrenceDate field flowing end-to-end (FB-01/FB-02)

**Flow:** Flow B (trigger schema + `Filter_Existing_Mapping`)
**Defect:** recurring meeting captures had no way to distinguish between different occurrence dates of the same series — the mapping table matched only on `SeriesMasterId`, meaning all occurrences of a series would resolve to the same existing page regardless of date.
**Fix applied (FB-01):** `Filter_Existing_Mapping` updated to match on both `SeriesMasterId` AND `OccurrenceDate` (`text_5`). `text_5` added to the Flow B trigger schema as `OccurrenceDate` (optional string field).
**Fix applied (FB-02):** `Send_an_HTTP_request_to_SharePoint` (mapping row write) updated to include `OccurrenceDate` in the JSON body.
**Fix applied (FB-03):** `C9B_Set_PageTitle` in the Topic updated to produce `Concatenate(Topic.MeetingTitle, " - ", Text(DateValue(Topic.DateContext), "d MMM yyyy"))` — uniform dated title format.
**Live-verified:** `OccurrenceDate`/`text_5` field confirmed flowing end-to-end.
**Full trace:** `session-2026-08-21-fb-progress-and-incidents.md`, `session-2026-08-21-fb04-build-and-getitems-mystery.md`.

---

## AMEND-2026-08-22-001 — Issue #1 completion: Date-based page matching (FB-04)

**Flow:** Flow B, `Filter_Pages_By_Title` + `Compose_RealExistingPageId`
**Defect:** Bug 9 workaround (`Compose_RealExistingPageId` blindly grabbing the first page in the section) meant that recapturing any recurring occurrence would always update the *first* page in the section regardless of date, rather than the correct occurrence-specific page.
**Fix applied (FB-04a):** `Filter_Pages_By_Title` `where` clause changed from `@equals(item()?['title'], outputs('Compose_MeetingTitleForPageMatch'))` to `@contains(item()?['title'], formatDateTime(triggerBody()?['text_5'], 'd MMM yyyy'))` — date-based matching.
**Fix applied (FB-04b):** `Compose_RealExistingPageId` changed from `first(outputs('Get_Pages_In_Section_Existing_Branch')...)` to `@if(greater(length(body('Filter_Pages_By_Title')), 0), first(body('Filter_Pages_By_Title'))?['id'], '')` — now consumes the filter's output.
**Fix applied (FB-05):** `Compose_SafePageTitle` and `Compose_SafePageTitle_OneOff` updated to append ` - [formatted date]` when `text_5` is present, so newly created pages have dated titles that FB-04's filter can match against.
**Live-verified:** full create → recapture cycle confirmed on 121 Simon/David series (16 Sep 2026 occurrence) and STDA series (30 Sep + 15 Oct 2026). Multiple occurrences confirmed creating separate dated pages under the same section. Recapture correctly identified existing page by date and appended update fragment without creating duplicate. Issue #1 fully closed.
**Full trace:** `session-2026-08-22-backlog-reduction-and-fb04-confirmed.md`, `session-2026-08-22-afternoon-addendum.md`.

## AMEND-2026-08-22-002 — `OutStatus` hardcoded to `"OK"` — differentiated to 6 values

**Flow:** Flow B, `Set_varOutStatus` + Topic `C11_Check_OutStatus`
**Defect:** `Set_varOutStatus` unconditionally set `"OK"` regardless of what actually happened — the agent always reported success even on partial failures or errors.
**Fix applied:** four new structural Compose actions added to capture mapping-write success/failure and section-match counts. `Set_varOutStatus` replaced with a six-value expression:
- `SUCCESS` — page action succeeded and mapping write confirmed (status 201)
- `PARTIAL_SUCCESS` — page action succeeded but mapping write failed
- `RECURRING_SETUP_REQUIRED` — recurring meeting but section resolution failed
- `SETUP_SECTION_NOT_FOUND` — no section URL resolved
- `SETUP_SECTION_AMBIGUOUS` — multiple sections matched
- `ERROR` — catch-all
**Companion change:** Topic `C11_Check_OutStatus` updated from `=Topic.OutStatus = "OK"` to `=Topic.OutStatus = "SUCCESS"`.
**Live-verified:** capture test immediately after publish confirmed `OutStatus = "SUCCESS"` and agent success message returned correctly.
**Full trace:** `session-2026-08-22-outstatus-differentiation.md`.

## AMEND-2026-08-22-003 — BadGateway on mapping row write — native connector replacement

**Flow:** Flow B, `Send_an_HTTP_request_to_SharePoint` (recurring) and `OF09a_—_Send_an_HTTP_request_to_SharePoint_(OneOff)` (one-off)
**Defect:** raw HTTP REST POST to SharePoint intermittently returned `502 BadGateway` after exhausting all retries, leaving the mapping row unwritten and causing subsequent recaptures to create duplicate pages.
**Fix applied:** both raw HTTP POST actions replaced with native SharePoint `Create item` connector actions (`PostItem` operation) — `Create_Mapping_Item_Recurring` and `Create_Mapping_Item_OneOff`. Native connector handles throttling and retries more gracefully. `Compose_MappingWriteSucceeded` and `Compose_MappingWriteSucceeded_OneOff` updated to reference new action names and check for status `201` (native connector returns 201, not 200). `HTTP_Update_SP_PageSelfUrl` and `OF09b_—_HTTP_Update_SP_PageSelfUrl_(OneOff)` URI references updated to use new action names for the new row's ID.
**Live-verified:** capture of "New Repeat Meeting" series confirmed mapping row written cleanly in 0.3s with green tick, no retries, `OutStatus = "SUCCESS"`, clickable OneNote link returned by agent.
**Full trace:** `session-2026-08-22-fa16-and-badgateway-fix.md`, `session-2026-08-22-badgateway-verification.md`.

## AMEND-2026-08-22-004 — FA43 coalescing gap: `IsRecurring` and `SeriesMasterId` always empty on multi-match path

**Flow:** Flow A (`PA - Resolve Meeting Selection`), `FA43_Respond_to_agent` response action
**Defect:** `isrecurring` and `seriesmasterid` fields in the response body were wired only to `FA19B_Compose_OutIsRecurring_Resolved` and `FA19C_Compose_OutSeriesMasterId_Resolved` respectively — the Resolved path only. All other response fields used `coalesce()` across multiple paths. Consequence: when a user selected a meeting from a multi-match candidate list, both fields were always returned empty, causing the Topic to incorrectly treat the selected meeting as a non-recurring one-off.
**Fix applied:** both fields updated to use `coalesce()` across all three paths:
```
isrecurring: coalesce(FA19B_Resolved, FA28A_Single, FA43A_Multi, '')
seriesmasterid: coalesce(FA19C_Resolved, FA28B_Single, FA43B_Multi, '')
```
**Live-verified:** FA16 defensive guard verification test (selecting meeting `2` from a 4-item candidate list) confirmed correct meeting identified and captured. FA43A/FA43B correctly remain empty on the initial multi-match response (no selection made yet); coalesce fallback ensures correct values on selection resolution.
**Full trace:** `session-2026-08-22-fa43-and-endofday.md`.

## AMEND-2026-08-22-005 — Section name sanitiser character gap

**Flow:** Flow B, `Compose_SafeSectionName`, `Compose_SafeSectionName_ExistingBranch`, `FB-F01_—_Compose_Input_MeetingTitle_(one-off)`
**Defect:** all three section-name sanitiser expressions were missing five characters from their `replace()` chain: `|`, `#`, `'`, `%`, `~`. These are invalid in OneNote section names and would cause `CreateSectionInNotebook` to fail with a BadRequest if any appeared in a meeting title.
**Fix applied:** five additional `replace()` pairs added to all three actions in both the `substring()` and `length()` halves of each expression. `\` (backslash) intentionally excluded — not a realistic character in Outlook meeting titles and caused Designer parser issues.
**Full trace:** `session-2026-08-22-backlog-reduction-and-fb04-confirmed.md`.

## AMEND-2026-08-22-006 — Link-format bug: `PageSelfUrl` returned instead of `oneNoteWebUrl`

**Flow:** Flow B, `Set_varOutputPageLink_Existing`
**Defect:** on the existing-page recapture path, `varOutputPageLink` was set to `varFinalExistingPageSelfUrl` (a REST API endpoint URL) rather than the clickable OneNote web URL, causing `C40001` authentication errors when users tried to open the returned link.
**Fix applied:** `Set_varOutputPageLink_Existing` value changed to `@first(body('Filter_Existing_Mapping'))?['PageWebUrl']`, reading the clickable web URL from the mapping row.
**Full trace:** `session-2026-08-22-backlog-reduction-and-fb04-confirmed.md`.

## AMEND-2026-08-22-007 — FA16 defensive guard

**Flow:** Flow A, `FA16_Compose_SelectedIndex`
**Defect:** if `varInSelectedNumber` contained a non-numeric string that bypassed the Topic's upstream `C6D_Check_Number` routing, `int()` would throw a runtime error rather than failing gracefully.
**Fix applied:** expression updated to add a round-trip guard: `not(equals(string(mul(int(if(equals(trim(...), ''), '1', trim(...))), 1)), trim(...)))` — if the value can't survive int→mul→string round-trip, falls back to `0`. Inner `'1'` fallback ensures `int()` always has a valid numeric string, avoiding eager-evaluation throws.
**Live-verified:** selecting meeting `2` from a 4-item candidate list confirmed correct, guard did not interfere with normal numeric selection.
**Full trace:** `session-2026-08-22-fa16-and-badgateway-fix.md`.

---

## Pre-log history (not numbered, see original docs)

Significant fixes made before this log began, documented in their own dated handover docs rather than under AMEND numbering:

- **P/N navigation root causes** (FA04 wrong trigger key, FA06/FA07 using `utcNow()` instead of `varDateContext`, FA15 not excluding P/N from numeric selection, `C6B_Check_N` unreachable, `C1_Set_DateContext` literal-string bug, missing `DateValue()` wrapping) — `2026-07-16-flow-a-pn-navigation-root-cause.md`, `2026-07-18-pn-navigation-topic-fixes.md`
- **Date-jump feature build** (`C6C_Check_Date`) and full UJ1–UJ5 validation pass — `2026-07-20-date-jump-feature-and-uj-validation.md`, `uj1`–`uj5-validation-record.md`
- **June 2026 Flow A/B bug-fixing campaign** — `varOutStatus` never set, FA09A filter/`runAfter` casing bugs, blank `Set variable` values across both flows, `Condition_IsRecurring` boolean-vs-string mismatch, null guard for all-day events (FA14), `int()` conversion guard (FA16) — see `handover-2026-06-*.md` series and `STRUCTURAL-FIXES-2026-06-21.md`
- **v5 UX rebuild** (numbered meeting list, P/N/date navigation, three-way match-count split) — `design-v5-ux-rebuild-2026-06-27.md`
