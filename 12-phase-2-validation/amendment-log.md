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

## Pending — to be logged once built

- **One-off existing-page resolution fix** (9-step build plan in `2026-07-31-oneoff-design-evidence-meetingid-column.md`) — not yet implemented as of 31 July. Log as `AMEND-2026-08-DD-NNN` once built and live-tested, covering: the `MeetingId`-keyed mapping lookup/write additions, the `Condition_Should_Create_Page` and `Set_varOutputPageLink_Existing` variable-reference changes, and the `Set varOutputPageSelfUrl Existing` value fix (closing out the Pattern 6 instance above).
- **`OutStatus` differentiation build** (`2026-07-27-flow-b-outstatus-trace.md`) — still the single highest-priority outstanding item per the 20 July gap-analysis doc. Not yet built.

---

## Pre-log history (not numbered, see original docs)

Significant fixes made before this log began, documented in their own dated handover docs rather than under AMEND numbering:

- **P/N navigation root causes** (FA04 wrong trigger key, FA06/FA07 using `utcNow()` instead of `varDateContext`, FA15 not excluding P/N from numeric selection, `C6B_Check_N` unreachable, `C1_Set_DateContext` literal-string bug, missing `DateValue()` wrapping) — `2026-07-16-flow-a-pn-navigation-root-cause.md`, `2026-07-18-pn-navigation-topic-fixes.md`
- **Date-jump feature build** (`C6C_Check_Date`) and full UJ1–UJ5 validation pass — `2026-07-20-date-jump-feature-and-uj-validation.md`, `uj1`–`uj5-validation-record.md`
- **June 2026 Flow A/B bug-fixing campaign** — `varOutStatus` never set, FA09A filter/`runAfter` casing bugs, blank `Set variable` values across both flows, `Condition_IsRecurring` boolean-vs-string mismatch, null guard for all-day events (FA14), `int()` conversion guard (FA16) — see `handover-2026-06-*.md` series and `STRUCTURAL-FIXES-2026-06-21.md`
- **v5 UX rebuild** (numbered meeting list, P/N/date navigation, three-way match-count split) — `design-v5-ux-rebuild-2026-06-27.md`
