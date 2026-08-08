# Bug 7 — Recurring meeting second-capture fails with BadRequest (sectionId/pageId format mismatch)

## Status: CONFIRMED, root cause identified, fix not yet applied

## Reported by David, 8 August 2026, live in production use (not a test session)

**Symptom:** Capturing a recurring meeting for the first time works correctly. Capturing the *same* recurring meeting series again the following week — i.e. the second, third, etc. occurrence — fails. Teams shows a generic error to the user (`FlowActionBadGateway`); the underlying flow run shows `Flow run failed`.

**Confirmed across two independent recurring meetings** on 8 August (different meetings, same failure point both times) — this is a systemic bug on this path, not a one-off fluke.

## Root cause

Traced live via Activity → Run results → Code view on the failing action.

Flow path taken: `Condition_IsRecurring` → True → `Condition_Mapping_Exists` → True (mapping found from last week) → `Condition_Should_Create_Page` → False (page already "exists" per the stored mapping) → `Condition_Is_Genuine_Existing_Page` → True → `Apply_to_each_Existing_Section` → **`Update_page_content_Existing_Branch` fails with BadRequest**.

Actual error body:
```json
{
  "error": {
    "code": 400,
    "message": "The section id given in the input is invalid. If a custom value was entered, please try selecting from the supplied values."
  }
}
```

Raw inputs on the failing call:
```
sectionId: 1-2175c4ee-3f51-48ad-b64a-6798d96d34bd
pageId:    1-1349b4ef932e4ec19799d3acc6c9fb68!20-2175c4ee-3f51-48ad-b64a-6798d96d34bd
```

**The section GUID is identical in both** (`2175c4ee-3f51-48ad-b64a-6798d96d34bd`) — this is genuinely the correct, same section. But the prefix differs: `sectionId` carries prefix `1-`, while the section reference embedded inside `pageId` (after the `!`) carries prefix `20-`. OneNote's `UpdatePageContent` API rejects the mismatch even though both values point at the same underlying section.

**Why this happens:** `sectionId` in this action is sourced from `items('Apply_to_each_Existing_Section')?['id']` — a section ID freshly returned by `GetSectionsInNotebook` (via `Get_Sections_Existing_Branch`), which returns IDs in `1-...` format. `pageId` is sourced from the stored `varOutputPageSelfUrl` (ultimately from the SharePoint mapping row's `PageSelfUrl` column, set at original page-creation time), which carries a page ID whose *embedded* section reference is in a different format (`20-...`). **Two different OneNote API surfaces return the section identifier in two different formats, and this branch mixes both formats in the same call.**

## Why this was never caught before now

Per `handover-2026-08-02-session6-two-paths-confirmed-working.md`, the recurring path testing that session only exercised **first capture** of a recurring meeting — i.e. `Condition_Mapping_Exists` → False → new mapping row created, new page created. That's a different code path entirely (`Create OneNote Page`, not `Update page content Existing Branch`) and doesn't touch this bug at all. The **second-capture / update-existing-page** path for recurring meetings had never actually been live-tested until David's real-world use today. This is a genuinely new finding, not a regression of anything previously "confirmed working."

## Relationship to other known issues

- **Distinct from Bug 5** (one-off recapture, empty `sectionId` string) — different symptom (populated-but-mismatched vs empty), different branch reached (`Condition_Is_Genuine_Existing_Page` → True/existing-page-update, vs → False/create-new-in-Bug-5's case), different root cause.
- **Related in spirit to the 6 August link-truncation finding** (`handover-2026-08-06-oneNoteWebUrl-link-truncation-future-build.md`) — both bugs trace back to the same underlying habit: storing/reusing a OneNote `PageSelfUrl`/ID value without validating exactly what format each OneNote API surface expects it in. Worth fixing both with this lesson in mind, though they are separate fixes in separate places.

## Fix direction (not yet applied — needs proper build session)

Rather than independently re-fetching the section by name and using *its* `id` for `sectionId`, derive `sectionId` directly from the **same source as `pageId`** — parse it out of the `pageId` string itself (the segment after `!`), so both values come from a source format OneNote's own `UpdatePageContent` call already expects to pair together correctly. Rough shape (expression to be properly built and tested in Designer, not typed freehand from memory):
```
substring(pageId, add(indexOf(pageId, '!'), 1), sub(length(pageId), add(indexOf(pageId, '!'), 1)))
```

This would mean `Get_Sections_Existing_Branch` / `Filter_Existing_Section_By_Name` / `Apply_to_each_Existing_Section` remain useful for *confirming the section exists at all* and locating the section to search within, but the `sectionId` actually passed into `UpdatePageContent` should come from parsing `pageId`, not from the freshly-fetched section object's own `id` field.

**Not yet confirmed:** exact behaviour of `indexOf`/`substring` needs testing against a real pageId value in this flow before committing to this expression — should be verified via Peek Code / a test run, not applied on faith.

## Priority

**High** — this blocks the single most common real-world usage pattern (a weekly recurring meeting, captured every week). Bug 5 (one-off recapture) and this bug are now both open; given this affects every recurring meeting's second-and-later capture, it likely has broader day-to-day impact than Bug 5.

## Status

**Root cause confirmed via two independent live failures, fix direction drafted, not yet built or tested.** Needs a dedicated session to build the corrected expression in Designer, verify via Peek Code, and live-test against a real recurring meeting's second capture before publishing.
