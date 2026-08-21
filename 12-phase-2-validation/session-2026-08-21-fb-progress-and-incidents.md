# Session log — #1 foundation built, live corruption incidents 4-5, session paused (21 August 2026)

## Status
**#1 (per-occurrence recurring pages): foundation complete and confirmed live. Final logic swap (FB-04) deliberately deferred to a calmer session.** Draft saved, clean Flow Checker, not published today.

## What was completed and confirmed live today

**Ref FB-TRIG-01** — `OccurrenceDate` (`text_5`) added to Flow B's trigger schema, optional. **Published.**

**Ref C10-01** — Topic's `C10_Call_FlowB_Create_Page` now sends `text_5: =Topic.DateContext`. **Published.**

**Live end-to-end confirmation**: real recurring capture of "121 Simon / David" (19 Aug 2026 occurrence) produced trigger body with `"text_5": "2026-08-19"` — correct ISO format, correct value, consistent with the embedded `text_3` title date. Confirmed via raw trigger inputs in Activity.

**Ref FB-01** — `Filter_Existing_Mapping` now matches on `SeriesMasterId` AND `OccurrenceDate`:
```
@and(equals(item()?['SeriesMasterId'],triggerBody()?['text_2']),equals(item()?['OccurrenceDate'],triggerBody()?['text_5']))
```
Built via "Edit in advanced mode" on the SharePoint filter action. Confirmed via Peek Code. **Saved draft, not yet published.**

**Ref FB-02** — mapping-write action (`Send_an_HTTP_request_to_SharePoint`) now includes `OccurrenceDate` in the POST body:
```json
"OccurrenceDate": "@{triggerBody()?['text_5']}"
```
Confirmed via Peek Code. **Saved draft, not yet published.**

**Ref FB-03** — Topic's `C9B_Set_PageTitle` simplified to a single uniform expression (recurring and one-off now both get meeting name + date, per explicit product decision for consistency):
```
=Concatenate(Topic.MeetingTitle, " - ", Text(DateValue(Topic.DateContext), "d MMM yyyy"))
```
**Saved in Topic draft, not yet published** (bundled with FB-01/02/04 for a single combined publish once complete).

**SharePoint schema** — `OccurrenceDate` column added to `RecurringMeetingSectionMap` (Single line of text, no default). Confirmed via screenshot.

## What's still open — Ref FB-04

Not started. `Compose_RealExistingPageId` (the Bug 9 "first page in section" workaround) still needs replacing with genuine date-based matching:
1. Wire up the currently-dead `Filter_Pages_By_Title` action to filter on `contains(item()?['title'], formatDateTime(triggerBody()?['text_5'], 'd MMM yyyy'))` (converts ISO `text_5` into the same `d MMM yyyy` display format used in page titles — this conversion function itself was never actually verified live today due to repeated interruption by corruption incidents; verify before building).
2. Point `Compose_RealExistingPageId` at `Filter_Pages_By_Title`'s output instead of the raw unfiltered page list.

No new connectors or actions required — both target actions already exist in the flow.

## Corruption incidents today (4th and 5th of the project's logged total)

Both hit the **same single action**, `OF05c — Set varFinalMatchCount (OneOff)`, losing its `value` field entirely (manifesting as a garbled `"tring"` function-not-defined error rather than a clean blank-value flag) — plus a 6th, broader hit affecting the same 26-action signature seen in incidents 1-3, immediately after adding/repositioning a temporary diagnostic Compose action (`Compose 1`) on the canvas.

**New pattern observed and worth escalating**: corruption appears more likely to strike around structural canvas changes (adding, moving, or repositioning actions) than during pure expression edits — the temporary `Compose 1` diagnostic action's addition/repositioning immediately preceded the last full 26-action wipe. This is a refinement to the existing "corruption is triggered by save/publish/settings events" understanding, not a contradiction of it.

All incidents recovered fully using `known-good-values-master-reference.md` — restored twice more today (once for the single `OF05c` action, once for the full 26). No data loss, no wrong-value restores.

## Recommendation

Escalate the Microsoft ticket submission further — this session alone contributed 2 additional distinct incidents (bringing the project total past what was previously logged), plus a specific new trigger pattern (structural canvas edits) worth adding as supporting evidence.

## Session close

Stopped deliberately after the second same-day full corruption event, on the reasoning that continued building risked outpacing our ability to verify what was genuinely fixed versus silently corrupted. Flow B: draft saved, Flow Checker clean, **not published**. Topic: FB-03 saved, **not published**. Nothing live-facing was changed today beyond the already-published `text_5` trigger field and Topic binding (both confirmed working before the corruption incidents began).

---
*Logged 21 August 2026. Cross-reference `design-amendment-2026-08-20-per-occurrence-recurring-pages.md` for the original #1 design, and `known-good-values-master-reference.md` for the restore data used twice this session.*
