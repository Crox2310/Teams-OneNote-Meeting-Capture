# Bug — `Create_Page_OneOff` fails with `BadRequest` for one-off meetings with a stale/broken SharePoint mapping — `varTargetSectionPagesUrl` never populated on this path

**Found:** 16 August 2026, during live UJ1 testing at work (real Teams conversation, not a synthetic test).
**Status:** OPEN — root cause identified, not yet fixed. Confirmed as a genuine design gap, not caused by any of yesterday's changes (though yesterday's title-fix work happened to be the first thing to newly touch this exact branch).

---

## Test case (use this to reproduce)

**Meeting:** "NH Performance Feedback - David, Simon & Jin Connect"
**Run:** 16 August 2026, 2:13 PM
**Trigger:** live Teams conversation — "capture meeting notes" → single match → agent proceeded automatically (normal UJ1 flow, no unusual user input).

## Symptom

```
Flow run failed. Action 'Create_Page_OneOff' failed: The section id given in the input
is invalid. If a custom value was entered, please try selecting from the supplied values.
```
Raw inputs confirm `sectionId` was genuinely **empty** at the point `Create_Page_OneOff` ran — not malformed, just blank.

## Root cause

Traced directly in Flow B's structure. The failure occurs on the `Condition_Should_Create_Page` = False branch → `Condition_Is_Genuine_Existing_Page` = False sub-branch — i.e. this meeting **already had a SharePoint `RecurringMeetingSectionMap` row** (so `varFinalPageDecision == 'PAGE_EXISTS'`), but the OneNote section that row points to no longer resolves as genuinely existing (`varOneNoteResolverResult` is not `ExistingMapping`/`ExistingSection`) — a stale or broken mapping.

On this specific sub-branch, `Create_Page_OneOff` runs using `variables('varTargetSectionPagesUrl')` as its `sectionId`. **That variable is only ever populated on this branch when `IsRecurring = true`**, via `Condition_Recurring_TargetSection` inside `Condition_Mapping_Exists`'s True branch. For a **one-off** meeting reaching this same stale-mapping scenario, nothing in the flow ever sets `varTargetSectionPagesUrl` — so by the time `Create_Page_OneOff` runs, it's blank, and OneNote correctly rejects the empty section reference.

**This is a genuine, previously-undiscovered gap in the one-off branch's logic**, not a corruption incident and not something introduced by 16 August's page-title work. It happened to be the first time this exact code path was exercised with fresh attention, because yesterday's `Create_Page_OneOff`/`Set_PageTitle_OneOff` work was investigating this same branch for an unrelated reason (title-setting) and found it was structurally narrower/rarer than assumed — but didn't test the `sectionId` gap specifically, since that test was deliberately deferred (see `handover-2026-08-16-oneoff-title-fix-built-unconfirmed.md`).

## Scope — this is not a one-off failure

Reviewing the flow's run history (`Activity` tab, last 7 days) shows a long, near-continuous run of **"Failed — An action failed. No dependent actions succeeded"** results going back through 15 and 16 August, interspersed with only occasional "Succeeded" runs. While not every failed run in that list has been individually confirmed to share this exact root cause, the volume and pattern strongly suggest **this same `Create_Page_OneOff` / empty-`sectionId` gap has likely been firing repeatedly across multiple real and test meetings**, not just this one case. Worth a systematic review of recent failed runs to confirm how many share this cause.

## Immediate unblock used (no flow changes)

The specific stale SharePoint mapping row for "NH Performance Feedback - David, Simon & Jin Connect" was identified and manually deleted from `RecurringMeetingSectionMap` (`https://jsainsbury.sharepoint.com/sites/coplt`), returning this meeting to the normal, already-proven `Create_OneNote_Page` path. This is a per-meeting workaround, not a fix — any other meeting with a stale mapping row will hit the same failure.

## Recommended fix

Add the missing `varTargetSectionPagesUrl` (and likely `varOneNoteResolverResult`) population logic to the one-off side of this branch, mirroring the recurring branch's `Condition_Recurring_TargetSection` pattern — i.e. when a one-off meeting's mapping exists but its section doesn't resolve as genuine, resolve/create the section fresh (similar to the existing `Get_Sections_OneOff`/`Filter_OneNote_Section_OneOff`/`Condition_Section_Exists_OneOff` chain used earlier in the flow for brand-new one-off captures) before falling through to `Create_Page_OneOff`.

## Recommended immediate next step

**Use this as the deliberate test case** for the stale-mapping edge-case testing flagged as outstanding in `handover-2026-08-16-oneoff-title-fix-built-unconfirmed.md`. Since this meeting has now had its stale mapping cleared, re-breaking it deliberately (e.g. manually inserting a mapping row pointing at a nonexistent/renamed section) would give a clean, repeatable way to build and verify the fix above, and to finally confirm whether `Set_PageTitle_OneOff` (built and unconfirmed as of 16 August) actually works once this blocking gap is closed.

---

**Status: OPEN. Confirmed real-world impact via live Teams testing, not just synthetic test data. Root cause identified with high confidence. Fix not yet designed in detail beyond the recommended approach above. Likely responsible for a meaningful share of this week's "Failed" runs in the Activity log — worth a broader review.**
