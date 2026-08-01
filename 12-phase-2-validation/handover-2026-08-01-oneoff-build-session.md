# Handover — 1 August 2026: One-Off Existing-Page Build Session (build complete, not yet live-tested)

## ⏭ START HERE NEXT SESSION

**Status: build complete. Nothing has been live-tested yet.** This session implemented the full 9-step plan from `2026-07-31-oneoff-design-evidence-meetingid-column.md`, plus two additional defects found and fixed mid-build that were not part of the original design. All changes are saved in Flow B (`PA - Resolve OneNote Meeting Section - v2 Clean Build`) but **not yet run live**.

**Next session should be a pure test session:**
1. Recapture a **brand-new one-off meeting** (no existing SharePoint mapping row) → confirm a new OneNote page is created, and confirm a new mapping row is written keyed on `MeetingId`.
2. Recapture the **same one-off meeting a second time** → confirm the existing page is found and updated (append), not duplicated, and confirm no `BadRequest`.
3. Recapture an **existing recurring meeting** (regression test) → confirm the recurring path still behaves exactly as before today's changes, since several shared actions were restructured.
4. Peek Code + raw run output at each new/changed action, per the established evidence-first standard for this project.

If all three pass, close out `handover-2026-07-30-oneoff-badrequest-live-trace-confirmation.md`'s status and log this build as `AMEND-2026-08-01-001` / `002` in `amendment-log.md` (drafted below, ready to paste in once confirmed live).

---

## What was built

### OF01–OF05 — one-off equivalents of the recurring mapping lookup

Inside `Condition_IsRecurring`'s False (one-off) branch, added:

- **OF01 — Filter_Existing_Mapping_OneOff** (Filter array): `from = body('Get_items')?['value']`, `where = equals(item()?['MeetingId'], triggerBody()?['text_4'])`
- **OF02 — Compose_ExistingPageSelfUrl_OneOff**: `if(greater(length(body('OF01...')), 0), first(body('OF01...'))?['PageSelfUrl'], '')`
- **OF03 — Compose_PageDecision_OneOff**: `if(not(empty(outputs('OF02...'))), 'PAGE_EXISTS', 'PAGE_NOT_FOUND')`
- **OF04 — Compose_Match_Count_OneOff**: `length(body('OF01...'))`
- **OF05a/b/c — three SetVariable actions**, writing into the **same shared variables** the recurring branch uses (`varFinalExistingPageSelfUrl`, `varFinalPageDecision`, `varFinalMatchCount`) — this is the crux of Option A (see design doc).

All five verified via Code view during the session.

### OF06 — `Condition_Should_Create_Page` repointed to shared variable

Was: `equals(outputs('Compose_PageDecision'), 'PAGE_NOT_FOUND')` (recurring-only, undefined for one-off).
Now: `equals(variables('varFinalPageDecision'), 'PAGE_NOT_FOUND')`.

Note the actual expression structure in this flow wraps the whole comparison in an outer `equals(..., true)` — that's the Designer's simple-mode representation of a full nested expression, not something we added.

### OF07 — `Set_varOutputPageLink_Existing` repointed

Was: `outputs('Compose_ExistingPageSelfUrl')` (recurring-only).
Now: `variables('varFinalExistingPageSelfUrl')`.

**Note:** this edit silently failed to save on the first attempt (Designer quirk — field appeared to accept the change but Code view showed the old value on re-check). Re-verify any single-field edits in this Designer via a fresh Code view pull before assuming they persisted; don't trust the field's visual state alone.

### OF08 — `Set_varOutputPageSelfUrl_Existing` placeholder replaced

Was: literal two-character string `"\"\""` (not a true empty string — a defect state slightly different from what the 30 July doc found, which showed a missing `value` key entirely; both produced the same broken outcome).
Now: `variables('varFinalExistingPageSelfUrl')`.

### Bonus fix (not in original design doc) — `Compose_ExistingPageId` blank inputs

Discovered mid-session: this action's `inputs` field was completely blank (`""`), which the Designer flagged as "Invalid parameters" once we started editing the surrounding actions. Per the 30 July trace doc, this action should extract the page ID from the self-URL. Fixed to:
```
last(split(variables('varOutputPageSelfUrl'), '/'))
```
This is load-bearing for `Update_page_content_Existing_Branch`'s `pageId` parameter — without it, even a correctly-populated `varOutputPageSelfUrl` wouldn't produce a usable page ID.

### OF09 — restructured from the design doc's original plan

**The design doc's original instruction (placing OF09 inside `Condition_IsRecurring`'s False branch) was found to be structurally wrong during the build** and was not followed as written. Reasoning:

- The existing recurring SharePoint-write actions (`HTTP_Update_SP_PageSelfUrl` etc.) actually live inside `Condition_Should_Create_Page`'s **True** branch, not inside `Condition_IsRecurring`'s branches — they need the newly-created page's URL, which doesn't exist yet at the point `Condition_IsRecurring` closes.
- `HTTP_Update_SP_PageSelfUrl`'s URI expression hard-references `Filter_Existing_Mapping` and `Send_an_HTTP_request_to_SharePoint` (both recurring-only actions) to resolve which SharePoint row to update. Once OF06 correctly routes one-off meetings into `Create_OneNote_Page`, they would hit this same shared action and fail, since neither referenced action ever runs on the one-off path.

**Actual structure built**, inside `Condition_Should_Create_Page`'s True branch, immediately after `Compose_PageSelfUrl_Created`:

- **OF09-Gate — Condition Is Recurring (SP Write)**: `equals(toLower(string(triggerBody()?['text'])), 'true')` (same trigger field and pattern as `Condition_IsRecurring` at the top of the flow)
  - **True branch:** the four original recurring actions (`HTTP_Update_SP_PageSelfUrl`, `Set_varPageAction_Created`, `Set_varOutputPageSelfUrl_Created`, `Set_varOutputPageLink_Created`) — **rebuilt from scratch**, not moved (see note below), with expressions matching the pre-existing working versions exactly.
  - **False branch:**
    - **OF09b-i — Condition Should Insert Mapping (OneOff)**: `equals(length(body('OF01_-_Filter_Existing_Mapping_OneOff')), 0)` — only insert a new row if OF01 found none.
      - **True → OF09a — Send an HTTP request to SharePoint (OneOff)**: inserts a new mapping row, `Title: "Mapping"`, `MeetingId: @{triggerBody()?['text_4']}`, `MeetingTitle`, `SectionPagesUrl`, `Status: "Active"`.
    - **After OF09b-i closes (both branches) → OF09b — HTTP Update SP PageSelfUrl (OneOff)**: always runs, updates `PageSelfUrl` on whichever row is correct via the same dual-purpose `if()` ID pattern as the recurring side, but resolving via `OF01`'s result or `OF09a`'s just-created row instead of the recurring equivalents.

**Note on the "rebuilt from scratch" decision:** the four recurring actions were initially *drag-moved* into `OF09-Gate`'s True branch rather than rebuilt. The move worked visually but silently dropped the `runAfter` property on the first-moved action, and this Designer's Settings tab has no way to manually reconfigure `runAfter`. Deleted and rebuilt all four fresh instead — slower but verifiably clean via Code view. **If moving actions between branches in this Designer again, verify `runAfter` via Code view immediately after — do not trust that a drag preserved it.**

**Other Designer quirks hit and worked around this session:**
- Typing a URI expression directly via the fx/expression editor can cause the entire field to become the expression (prefixing a stray `@`) rather than treating it as literal text with an embedded `@{...}`. Workaround: type the literal path text first as plain text, then insert only the `if(...)` portion via the dynamic content picker so it becomes an inline `@{...}` chip.
- Copy-pasting Code view content into a field literally (including the `"parameters/uri": ` JSON key label) is easy to do by accident when working from a previous Code view screenshot — always re-verify the actual field content, not just that something was typed.

### OF10 — new defect found mid-session, not in original design doc

**`Condition_Mapping_Exists`'s True branch was previously unreachable for one-off meetings** (per the 30/31 July docs) because `varFinalMatchCount` was never populated on that path. **OF05c fixes exactly that gap** — which means this previously-dead branch became live for one-off meetings for the first time as a side effect of today's build.

Found: `Set_varTargetSectionPagesUrl_ExistingMapping` inside that branch contained an `if()` expression with **both arms pointing at the same recurring-only action** (`Filter_Existing_Mapping`) — an apparent unfinished stub from an earlier session, never triggered before because the branch was unreachable. Left as-is, this would likely throw a runtime error (referencing an action that never ran) the first time a one-off meeting with an existing mapping row hit this path — i.e., on any one-off meeting's second-or-later recapture, once OF09 starts successfully writing rows.

**Fix applied:**
```
if(equals(toLower(string(triggerBody()?['IsRecurring'])), 'true'),
   first(body('Filter_Existing_Mapping'))?['SectionPagesUrl'],
   first(body('OF01_-_Filter_Existing_Mapping_OneOff'))?['SectionPagesUrl'])
```

**Open dependency to verify in testing:** this assumes the SharePoint mapping list's `SectionPagesUrl` column gets populated correctly for one-off rows by OF09a/OF09b, the same way it does for recurring rows. Worth checking directly in the SharePoint list during the test session, not just inferring from the flow logic.

---

## Draft amendment log entries (paste into `amendment-log.md` once live-tested and confirmed)

### AMEND-2026-08-01-001 — One-off existing-page resolution (full build per 31 July design)

**Flow:** Flow B
**Defect:** no mechanism existed to resolve a specific existing OneNote page for a one-off meeting on recapture; `Condition_Should_Create_Page` and `Set_varOutputPageLink_Existing` referenced recurring-only action outputs, undefined for one-off meetings; `Compose_ExistingPageId` had blank inputs; `Set_varOutputPageSelfUrl_Existing` held a non-functional placeholder value.
**Fix applied:** OF01–OF09 per `2026-07-31-oneoff-design-evidence-meetingid-column.md`, restructured at build time (see `handover-2026-08-01-oneoff-build-session.md` for the actual placement, which diverged from the original design doc's OF09 location).
**Live-verified:** [pending next session]

### AMEND-2026-08-01-002 — `Condition_Mapping_Exists` True branch stub completed (OF10)

**Flow:** Flow B
**Defect:** `Set_varTargetSectionPagesUrl_ExistingMapping`'s IsRecurring-aware `if()` had both arms referencing the same recurring-only action, made newly reachable for one-off meetings as a side effect of AMEND-2026-08-01-001.
**Fix applied:** false/one-off arm repointed to `OF01_-_Filter_Existing_Mapping_OneOff`.
**Live-verified:** [pending next session]

---

## Status

**Build complete, unverified live.** Next session: test session only, per the three scenarios at the top of this doc. Do not begin further feature work until the recurring-path regression test has passed — several shared actions were touched today.
