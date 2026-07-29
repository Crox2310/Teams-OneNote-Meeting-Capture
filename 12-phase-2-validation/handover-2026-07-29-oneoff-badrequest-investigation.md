# Session Handover — 2026-07-29 (Flow B One-Off BadRequest Root Cause Investigation)

## Status: Root cause identified and confirmed with hard evidence. NO FIX APPLIED YET — investigation only. Flow currently unpublished/untouched beyond one reverted attempt (see "Fix attempts made and reverted" below).

## Context / how this session started

Session began as a live-debugging follow-on from the 27 July `OutStatus` prep session. While testing in the Copilot Studio Test panel with a **one-off (non-recurring) meeting**, the agent surfaced:

```
Error Message: The flow 'PA - Resolve OneNote Meeting Section - v2 Clean Build' failed to run with response code 'BadGateway', error code: NotSpecified.
```

This generic `BadGateway` in Copilot Studio turned out to be masking a real `BadRequest` inside Flow B (`PA - Resolve OneNote Meeting Section - v2 Clean Build`) at the `Update page content Existing Branch` action.

## Important process note for next session

This session went through a period of **confused, contradictory conclusions** caused by reasoning from static Designer JSON and partial screenshots rather than anchoring to the actual executed run in Activity. Two false leads were chased and corrected:
1. Initially concluded the fix was a simple variable-name swap in `Compose ExistingPageId` (from `varFinalExistingPageSelfUrl` to `varOutputPageSelfUrl`). This was applied, tested, and did **not** fix the issue — it just changed the symptom (empty `pageId` instead of literal unparsed formula text).
2. Then incorrectly concluded `Set_varTargetSectionPagesUrl_ExistingMapping` / `Set_varOneNoteResolverResult_ExistingMapping` had "succeeded live" based on a misread screenshot; corrected this once the real Activity trace was checked, confirming these were actually **Skipped** (`ActionConditionFailed`) for a one-off meeting.

**Lesson for next session: always confirm status (Succeeded vs Skipped) and read real values from the Activity trace's Run Results, never infer from Designer canvas structure or JSON alone.** The flow has accumulated multiple parallel implementations (original/legacy, recurring-specific, one-off-specific) across incremental rebuilds, and only the live Activity trace reliably shows which one actually executed for a given test.

## Confirmed true execution path for a one-off meeting (from real Activity trace, run at 22:33–22:34 on 29 July)

1. `Get items` (SharePoint `RecurringMeetingSectionMap` list — full unfiltered pull)
2. `varFinalExistingPageSelfUrl` — **Initialize only, value `null`. Never Set anywhere in the executed path. Confirmed dead.**
3. `varFinalPageDecision`, `varFinalMatchCount`, `varOutStatus`, `varOutputPageLink`, `varOutputPageSelfUrl` — various initializes
4. `Condition IsRecurring` → **False** (correct — test meeting was one-off)
5. `FB-F01 — Compose Input MeetingTitle (one-off)` → `Get Sections OneOff` → `Filter OneNote Section OneOff` → `Compose Section Match Count OneOff` → `Condition Section Exists OneOff` → **True** → `For each 1` (1 of 4) → `Set varTargetSectionPagesUrl OneOff Exists` / `Set varOneNoteResolverResult Exists OneOff`
6. `Condition Mapping Exists` (tests `varFinalMatchCount > 0`) → **True** branch executes: `Compose PageRoute Exists` → `Compose Branch Result` → `Set varTargetSectionPagesUrl ExistingMapping` → `Set varOneNoteResolverResult ExistingMapping`
   - **Important:** despite the name, this True branch's actions do **not** depend on `Filter_Existing_Mapping` at all in the real executed run — confirmed via their own Code view (`runAfter: Compose_Branch_Result`, no reference to `Filter_Existing_Mapping` body). The earlier theory that they read from `Filter_Existing_Mapping` was wrong; `Filter_Existing_Mapping` (the `SeriesMasterId`-based query, which only exists inside `IsRecurring`'s True branch) was confirmed **Skipped** for this run.
7. `Condition Should Write Mapping` → True → `Send an HTTP request to SharePoint` (writes a new mapping row)
8. **`Condition Should Create Page` → False** (page already exists) → `Set varPageAction ExistsNoCreate` → **`Set varOutputPageSelfUrl Existing`** (`value: @variables('varFinalExistingPageSelfUrl')` — reads the dead null variable from step 2) → `Compose UpdateHtmlFragment` → `Compose ExistingPageId` (`@last(split(variables('varOutputPageSelfUrl'), '/'))` → evaluates to empty string because its source is null)
9. `Condition Is Genuine Existing Page` → True → `Get Sections Existing Branch` → `Filter Existing Section By Name` → `Apply to each Existing Section` (1 of 1) → **`Update page content Existing Branch` fails: `BadRequest`, `pageId: ""`**

## Root cause (confirmed)

**`Set varOutputPageSelfUrl Existing` (inside `Condition Should Create Page`'s False branch) sets `varOutputPageSelfUrl` from `variables('varFinalExistingPageSelfUrl')`.** That source variable is Initialized to `null` at the very top of the flow and is **never Set anywhere in the one-off execution path**. It IS correctly Set inside `IsRecurring`'s True (recurring) branch, via a completely separate sub-chain (`Compose Input SeriesMasterId` → `Filter Existing Mapping` → `Compose ExistingPageSelfUrl` → `varFinalExistingPageSelfUrl 1`) — but that entire chain is skipped for one-off meetings.

**Net effect:** for a one-off meeting where the page already exists, nothing in the flow ever populates the *specific existing page's* self URL. The mapping-exists logic (`Condition Mapping Exists` True branch, step 6 above) sets `varTargetSectionPagesUrl` and `varOneNoteResolverResult` — useful for section-level identity — but **never sets a page-level identifier**. This looks like a genuine, previously-undiscovered gap in the one-off branch's build, not a wiring mistake — the equivalent of the recurring branch's mapping-based page lookup was never built for one-off meetings.

## What's still unknown / needs deciding next session

1. **Where should the existing page's self URL actually come from for a one-off meeting?** No candidate step currently exists. Options to evaluate:
   - Extend `Get items` (SharePoint mapping table) lookup logic used in step 6 to also capture a `PageSelfUrl` field from the matched row (if the `RecurringMeetingSectionMap` list has one — needs checking; the recurring branch's `Compose ExistingPageSelfUrl` reads `first(body('Filter_Existing_Mapping'))?['PageSelfUrl']`, implying the list does have this column).
   - Build a one-off-specific filter against `Get_items`'s already-pulled data (by meeting title, since there's no `SeriesMasterId` for one-off meetings) and Set `varFinalExistingPageSelfUrl` from the matched row's `PageSelfUrl`, mirroring the recurring branch's pattern.
2. Whether `Condition Mapping Exists`'s True branch (step 6) is the right place to add this, or whether it needs its own one-off-specific filter step similar to `Filter_Existing_Mapping`.
3. The broader question raised earlier this session (not yet resolved): whether the original/legacy `varFinalExistingPageSelfUrl`-based chain (steps 1–2) is dead code that should be removed, or still serves some purpose not yet identified.

## Fix attempts made and reverted this session

- Changed `Compose ExistingPageId`'s expression from referencing `varFinalExistingPageSelfUrl` to `varOutputPageSelfUrl` (via Designer fx editor). This was tested and did not resolve the BadRequest — `pageId` came through empty rather than as unparsed text, confirming `varOutputPageSelfUrl` was *also* empty (since it too derives from the dead `varFinalExistingPageSelfUrl`). **This change may still be present in the live Designer — needs checking and likely reverting or superseding at the start of next session**, since the real root cause is upstream of both variables.

## Data captured this session

Full Code view JSON was captured for every action in Flow B, screenshot by screenshot, covering: the full `Condition Mapping Exists` node (both branches), `Condition Should Create Page` (both branches), `Condition Is Genuine Existing Page` (both branches), `Condition IsRecurring` (both branches, including the full recurring sub-chain and the one-off sub-chain), `Respond to the agent`'s full output schema, and the trigger schema. This was intended to be compiled into a canonical structural reference but the session ended before compilation — **worth doing at the start of next session** so future debugging doesn't need to re-screenshot the whole flow.

## Recommended next steps, in order

1. Check whether the `Compose ExistingPageId` expression change (variable swap) is still live in Designer; decide whether to revert it or leave it (it's neutral either way until the real fix lands, since both source variables are currently empty for one-off meetings).
2. Confirm the `RecurringMeetingSectionMap` SharePoint list schema — specifically whether it has a `PageSelfUrl` column (strongly implied by the recurring branch's usage) and whether one-off meetings' pages get written there at all currently (check `Set_varOutputPageSelfUrl_Created` / `Set_varOutputPageSelfUrl_Existing` and the `HTTP Update SP PageSelfUrl` action to see if one-off page creation ever updates this table).
3. Design and build the missing one-off page-lookup step (see "What's still unknown" above) — likely a new filter + compose + Set action sequence mirroring the recurring branch's pattern, feeding `varFinalExistingPageSelfUrl` (or a new dedicated variable) for one-off meetings.
4. Retest with the same one-off meeting scenario used this session.
5. Once fixed, log as a new amendment (next number after AMEND-2026-07-27-002) in `11-baseline/amendment-log.md`, and update `living-audit.md` per the standing caution already noted there.
6. Revisit the `OutStatus` six-value build (original goal, still not started) once this defect is closed — it was blocked on exactly this kind of unresolved page-existence logic.
