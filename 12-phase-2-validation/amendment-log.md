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

## Pending — to be logged once built

- **`OutStatus` differentiation build** (`2026-07-27-flow-b-outstatus-trace.md`) — still an outstanding item per the 20 July gap-analysis doc. Not yet built.
- **Recurring-branch title-set race condition fix** (retry/poll approach, replacing the abandoned Delay-based mitigation) — see AMEND-2026-08-16-003's "known follow-up issue."
- **One-off branch title fix live confirmation** — see AMEND-2026-08-16-004; needs a deliberately-constructed stale-mapping test.

---

## Pre-log history (not numbered, see original docs)

Significant fixes made before this log began, documented in their own dated handover docs rather than under AMEND numbering:

- **P/N navigation root causes** (FA04 wrong trigger key, FA06/FA07 using `utcNow()` instead of `varDateContext`, FA15 not excluding P/N from numeric selection, `C6B_Check_N` unreachable, `C1_Set_DateContext` literal-string bug, missing `DateValue()` wrapping) — `2026-07-16-flow-a-pn-navigation-root-cause.md`, `2026-07-18-pn-navigation-topic-fixes.md`
- **Date-jump feature build** (`C6C_Check_Date`) and full UJ1–UJ5 validation pass — `2026-07-20-date-jump-feature-and-uj-validation.md`, `uj1`–`uj5-validation-record.md`
- **June 2026 Flow A/B bug-fixing campaign** — `varOutStatus` never set, FA09A filter/`runAfter` casing bugs, blank `Set variable` values across both flows, `Condition_IsRecurring` boolean-vs-string mismatch, null guard for all-day events (FA14), `int()` conversion guard (FA16) — see `handover-2026-06-*.md` series and `STRUCTURAL-FIXES-2026-06-21.md`
- **v5 UX rebuild** (numbered meeting list, P/N/date navigation, three-way match-count split) — `design-v5-ux-rebuild-2026-06-27.md`
