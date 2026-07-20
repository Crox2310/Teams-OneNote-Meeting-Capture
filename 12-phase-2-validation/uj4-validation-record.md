# UJ4 Validation Record — First-Time Recurring Setup

## User Journey
UJ4 — First-time recurring setup

---

## Purpose
Validate that the Meeting Capture agent correctly:

- Identifies a recurring Outlook meeting (`IsRecurring = true`) with **no** existing SharePoint mapping
- Creates a new OneNote section for the series, named consistently with the one-off naming convention
- Writes a new `RecurringMeetingSectionMap` row in SharePoint recording the series-to-section mapping
- Creates the page within the newly created section
- Returns a valid page link and success response

---

## Test History

### Status prior to 2026-07-19: never formally passing
Same root cause as UJ3 — `Condition_IsRecurring` never evaluated True, so the entire recurring-setup path (mapping-exists check, mapping-write, section-creation) was dead code for the entire life of the project until fixed on 2026-07-19.

Two additional, independent bugs were found and fixed as part of getting this journey working for the first time:

1. **`Condition_Should_Write_Mapping` duplicated its parent condition's expression** (`greater(varFinalMatchCount, 0)`), making the mapping-write branch unreachable even once `Condition_IsRecurring` was fixed. Fixed by changing the condition to check `IsRecurring` directly. See `2026-07-18-flow-b-mapping-exists-trace.md`, Finding 1.
2. **No section-creation logic existed** in the "mapping doesn't exist" branch — `varTargetSectionPagesUrl` was never populated for a genuinely new series, causing `Create OneNote Page` to fail with "The section id given in the input is invalid." Fixed by building a new action chain (`Get Sections Recurring` → `Filter OneNote Section Recurring` → `Compose Section Match Count Recurring` → `Condition Section Exists Recurring`, with True/False branches to reuse or create a section) mirroring the existing one-off logic. Built 2026-07-18/19.
3. **`Compose_SafeSectionName` produced a different name than the one-off path's `FB-F01`** (missing the `"Mtg - "` prefix, trimmed trailing whitespace, different character-replacement set, different truncation length), causing a duplicate section to be created instead of matching the existing one-off-style name. Fixed by rewriting `Compose_SafeSectionName` to be an exact mirror of `FB-F01`'s logic. Fixed 2026-07-19.

### Validation (2026-07-19) — PASS
Tested by capturing "Supply Chain Weekly Release Call" after clearing all prior mapping rows and duplicate/incorrectly-named sections from earlier debugging.

**Result:**
- `Condition_IsRecurring` → True
- `Condition Mapping Exists` → False (no existing mapping, correctly detected)
- `Filter OneNote Section Recurring` found no match (correct — nothing existed yet)
- `Create Section Recurring` created a new section named `"Mtg - Supply Chain Weekly Release Call "` (matching the one-off naming convention exactly, including the trailing space from the source title)
- `Condition_Should_Write_Mapping` → True → `Send_an_HTTP_request_to_SharePoint` succeeded (HTTP 201, single new mapping row, `Id: 20`+)
- SharePoint list confirmed exactly one clean `Mapping` row, no duplicates
- OneNote confirmed exactly one correctly-named section
- Agent response: full success message with valid page link

---

## Current Validation Outcome

✅ **PASS — validated 2026-07-19, first successful validation of this journey**

---

## Contract Validation

### Flow B Contract
✅ `Condition_IsRecurring` correctly evaluates True
✅ `Condition Mapping Exists` correctly evaluates False for a genuinely new series
✅ Section creation chain correctly creates a new, correctly-named section and populates `varTargetSectionPagesUrl`
✅ `Condition_Should_Write_Mapping` correctly evaluates True and writes exactly one mapping row
✅ `OutStatus = "OK"` returned

---

## Observations

- This is the journey with the most compounded historical bugs of any of the five — three independent, layered defects had to be found and fixed before it worked even once.
- Naming consistency between the one-off (`FB-F01`) and recurring (`Compose_SafeSectionName`) section-naming logic is now critical and shared — if either is changed in future, the other must be updated to match, or duplicate sections will start being created again silently.
- Testing this journey destructively consumes state (creates a mapping + section) — subsequent tests of the same meeting will exercise UJ3 (reuse), not UJ4 (create), unless the mapping/section is manually cleared first.

---

## Baseline Status

✅ **BASELINED — 2026-07-19**

UJ4 is approved as the authoritative implementation for:

> First-time capture of a recurring meeting series with no existing SharePoint mapping

---

## Next Step

All five original user journeys (UJ1–UJ5) are now validated. Remaining work is documentation (living audit update) and validation of the newly built date-jump navigation feature, which extends beyond the original UJ1–UJ5 scope.
