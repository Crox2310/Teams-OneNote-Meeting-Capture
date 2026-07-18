# Flow B (PA - Resolve OneNote Meeting Section) — Condition Mapping Exists Trace

**Date:** 2026-07-18
**Method:** Full peek-code trace of Flow B from the trigger schema through `Condition IsRecurring` and `Condition Mapping Exists`, completing the last unverified section identified in `2026-07-18-flow-b-back-half-trace.md`.
**Status:** One significant, currently-live bug confirmed (Finding 1 below) — the SharePoint mapping-write path for brand-new recurring meeting series is structurally unreachable. Two lower-priority findings also confirmed. Flow B is now fully traced end to end.

---

## 1. Structural map

```
Trigger (text=IsRecurring, text_1=MeetingTitle, text_2=SeriesMasterId, text_3=PageHtml, text_4=MeetingId)
↓
Get_items (SharePoint Get_items — reads RecurringMeetingSectionMap list)
↓ variable init: varFinalExistingPageSelfUrl, varTargetSectionPagesUrl, varOneNoteResolverResult, varPageAction, etc.
↓
Condition_IsRecurring  (=toLower(string(triggerBody()?['text'])) = 'true' — correct field reference)
├─ True (recurring)
│  ├─ Compose_Input_SeriesMasterId, Compose_Input_MeetingTitle
│  ├─ Filter_Existing_Mapping   (filters Get_items by SeriesMasterId)
│  ├─ Compose_ExistingPageSelfUrl, Compose_PageDecision, Compose_Match_Count
│  └─ Set varFinalExistingPageSelfUrl_1 / varFinalPageDecision_1 / varFinalMatchCount_1
│
└─ False (one-off)
   ├─ FB-F01 (Compose Input MeetingTitle, one-off — fixed 2026-07-18, see back-half doc)
   ├─ Get_Sections_OneOff, Filter_OneNote_Section_OneOff, Compose_Section_Match_Count_OneOff
   └─ Condition_Section_Exists_OneOff
      ├─ True: For_each_1 → Set varTargetSectionPagesUrl_OneOff_Exists / varOneNoteResolverResult_Exists_OneOff = "ExistingSection"
      └─ False: Create_Section_OneOff → Set varTargetSectionPagesUrl_OneOff_Created / varOneNoteResolverResult_Created_OneOff = "CreatedSection"

↓ (both branches converge, runAfter varPageAction)

Condition Mapping Exists   (=greater(int(if(empty(varFinalMatchCount),'0',varFinalMatchCount)),0) = true)
├─ True — a SharePoint mapping row already exists for this series
│  ├─ Compose_PageRoute_Exists, Compose_Branch_Result = "EXISTS"
│  ├─ Set_varTargetSectionPagesUrl_ExistingMapping  (see Finding 2)
│  └─ Set_varOneNoteResolverResult_ExistingMapping = "SHAREPOINT_MAPPING_EXISTS"
│
└─ False — no mapping row found
   ├─ Compose_Branch_Result_NoMatch = "CREATE_REQUIRED"
   ├─ Compose_SectionDisplayName = triggerBody()?['text_1']  (MeetingTitle)
   ├─ Compose_SafeSectionName  (same nested-replace sanitisation pattern as FB-F01, applied to the section name)
   ├─ Condition_Should_Write_Mapping  ← see Finding 1, this branch is dead
   │  ├─ True: Send_an_HTTP_request_to_SharePoint  (POST new row: Title="Mapping", SeriesMasterId, MeetingTitle, SectionPagesUrl, Status="Active")
   │  └─ False: 0 actions
   ├─ Compose_IgnoreSeriesMasterId = "''"  (vestigial no-op, harmless)
   └─ Compose_PageRoute_CreateRequired = "PAGE_NOT_FOUND_ROUTE"

↓ (both branches converge — this is where the previously-traced back half picks up)

Condition Should Create Page → ... → Respond to the agent
```

The `Condition Should Create Page` → `Respond to the agent` portion (already documented in `2026-07-18-flow-b-back-half-trace.md`) was re-confirmed in this batch with no new discrepancies — matches exactly.

---

## 2. Findings

### Finding 1 🔴 — SharePoint mapping is never written for a genuinely new recurring meeting series
`Condition_Should_Write_Mapping` sits inside the **False** branch of the outer `Condition Mapping Exists` — i.e. we only reach it once we already know `varFinalMatchCount` is *not* greater than 0. Its own expression is:

```
greater(int(if(empty(variables('varFinalMatchCount')), '0', variables('varFinalMatchCount'))), 0) == true
```

This is the **exact same condition** as the outer check we just failed. `varFinalMatchCount` isn't reassigned anywhere in between, so this inner condition can never evaluate `true` in the context it's placed — confirmed directly on the canvas, where the False side of `Condition_Should_Write_Mapping` shows "0 Actions" and the True side (containing `Send_an_HTTP_request_to_SharePoint`, the actual SharePoint row write) is consequently unreachable.

**Effect:** the first time a new recurring meeting series is captured, the code clearly intends to write a new `RecurringMeetingSectionMap` row (`Compose_SafeSectionName` sanitises a section name specifically for this purpose, and the HTTP POST body is fully built out) — but that write never actually fires. The series never gets a mapping row, so every future capture of that same recurring meeting will also find no mapping and keep falling through to the same dead branch. Recurring-meeting section mapping can never self-establish for a new series under the current logic.

This looks like a copy-paste error — the condition was very likely meant to check something else entirely, most plausibly whether the meeting is recurring at all (`equals(toLower(string(triggerBody()?['text'])), 'true')`, mirroring `Condition_IsRecurring`'s own check), since one-off meetings don't need a SharePoint mapping row and already have their own `Create_Page_OneOff` path. That's a hypothesis based on the surrounding code's evident intent, not a confirmed original design — worth confirming the intended behaviour before changing it, but the current state (the write never fires under any input) is very unlikely to be intentional.

### Finding 2 🟡 — Redundant branching + a masked field-name bug in `Set_varTargetSectionPagesUrl_ExistingMapping`
```
if(equals(toLower(string(triggerBody()?['IsRecurring'])), 'true'),
   first(body('Filter_Existing_Mapping'))?['SectionPagesUrl'],
   first(body('Filter_Existing_Mapping'))?['SectionPagesUrl'])
```
Both branches of this inline `if` return the identical expression, so the condition is functionally a no-op — harmless on its own. But the condition itself references `triggerBody()?['IsRecurring']`, which doesn't exist in the trigger schema (the actual field is `text`, with `IsRecurring` only as its display title) — the same field-name mismatch pattern already fixed in Flow A's `FA04` this session. Right now this is masked by Finding 2's redundant branching (both outcomes are identical regardless of which way the broken condition resolves), but if someone "cleans up" the redundant `if` later without also fixing the field reference, this would become a live bug. Worth fixing both together.

### Finding 3 🟡 — Two empty `SetVariable` actions currently showing "Invalid parameters" in Designer
`Set_varPageAction_ExistsNoCreate` (`value: ""`) and `Set_varOutputPageSelfUrl_Existing` (`value: ""`) sit at the start of `Condition Should Create Page`'s False branch, ahead of `Condition Is Genuine Existing Page`. Both are currently flagged with a live red "Invalid parameters" validation error in Designer — not a cosmetic banner like the "Flow not found" ones, but an actual parameter validation failure. They appear to be vestigial placeholders: the real assignment of `varPageAction`/`varOutputPageSelfUrl` for this path happens later, inside `Condition_Is_Genuine_Existing_Page`'s own True/False branches (`Set_varPageAction_UpdatedAppend`, `Set_varOutputPageLink_Existing`, `Set_varOutputPageLink_Created_OneOff`). Since the 2026-07-18 live test only exercised brand-new page creation (`Condition Should Create Page`'s True branch), this False-branch path — updating an existing page, or one-off creation — has not yet been live-tested since these fixes. Worth testing that path specifically, and removing or fixing these two empty SetVariable actions regardless, since "Invalid parameters" is Designer telling us something is genuinely wrong, not just stale.

---

## 3. Recap of back-half findings (from `2026-07-18-flow-b-back-half-trace.md`, still open)

- `OutStatus` hardcoded to `"OK"`, no error branch.
- Two different OneNote connections (`shared_onenote` vs `shared_onenote-1`) used for page creation.
- Duplicate output fields (`outtargetsectionpagesurl`/`outfinaltargetsectionpagesurl`, `outresolverresult`/`outonenoteresolverresult`) — cosmetic only.

---

## 4. Recommended priority

1. **Finding 1** is the highest-priority item found across this entire debugging project to date — it silently breaks recurring-meeting mapping creation for any new series, with no error or symptom visible to the user (the flow completes "successfully," just without ever writing the mapping). Recommend confirming intended behaviour, then fixing `Condition_Should_Write_Mapping`'s expression.
2. **Finding 3** should be verified with a live test of the "existing page" / "one-off create" path specifically, since it wasn't exercised by the 2026-07-18 end-to-end test (which only hit brand-new page creation).
3. **Finding 2** and the back-half findings are lower priority — safe to batch together whenever Flow B next gets touched.

---

## 5. Flow B status

With this trace, Flow B has now been fully peek-code-verified end to end (trigger → `Condition_IsRecurring` → `Condition Mapping Exists` → `Condition Should Create Page` → `Respond to the agent`), matching the same rigor already applied to Flow A and the Topic. No further unverified sections remain.
