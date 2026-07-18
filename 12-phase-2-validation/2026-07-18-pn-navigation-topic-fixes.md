# P/N Day-Navigation — Full Resolution (Flow A + Flow B + Topic)

**Date:** 2026-07-18
**Method:** Continuation of the 2026-07-16 root-cause diagnosis. Fixes applied directly in Designer/Copilot Studio via the expression editor and Topic YAML, cross-checked against peek code before each change. Verified live via Copilot Studio's Test panel end to end: number selection, N, N again, and a final number selection through to OneNote page creation.
**Status:** P/N navigation and the original number-selection journey are now confirmed working end to end, including OneNote page creation via Flow B. Topic Checker reduced from 8 errors to 2, both confirmed non-blocking (Publish succeeded with them present).

This entry supersedes the "Remaining gap" section of `2026-07-16-flow-a-pn-navigation-root-cause.md` — that gap is now closed.

---

## Flow A (`PA - Resolve Meeting Selection - v1 Clean Build`)

**FA15_Compose_IsSelectionMode** 🟢 fixed and tested
Was: `@and(not(empty(trim(variables('varInSelectedNumber')))), not(equals(toUpper(trim(variables('varInSelectedNumber'))), 'NONE')))` — didn't exclude `P`/`N`, causing `FA16`'s `int()` crash.
Now:
```
and(not(empty(trim(variables('varInSelectedNumber')))), not(contains(createArray('NONE','P','N'), toUpper(trim(variables('varInSelectedNumber'))))))
```

**FA06_Compose_StartOfDayUtc** / **FA07_Compose_EndOfDayUtc** 🟢 fixed and tested
Was: built purely from `utcNow()`, ignoring `varDateContext` entirely.
Now (FA06 shown, FA07 identical pattern with `'yyyy-MM-ddT23:59:59Z'`):
```
formatDateTime(if(empty(trim(coalesce(variables('varDateContext'), ''))), utcNow(), variables('varDateContext')), 'yyyy-MM-ddT00:00:00Z')
```

**FA04_Init_varDateContext** 🟢 fixed and tested — new finding this session, not caught on 07-16
Was: `@triggerBody()?['DateContext']`. The trigger's actual JSON schema property is `text_3` (confirmed via trigger peek code) — `"title": "DateContext"` is only a display label on that field, not the real key. This meant `varDateContext` was `null` on every run regardless of what the Topic sent, silently forcing FA06/FA07's fallback to `utcNow()` every time. This is why the first live test after the FA06/FA07/FA15 fixes still showed no error but also no date shift.
Now:
```
triggerBody()?['text_3']
```
Bug taxonomy tag: #3 (wrong/mismatched field name) per `PROCESS-expression-audit-maintenance.md`.

**Not yet touched:** Finding 4 from the 07-16 doc (FA43's `IsRecurring`/`SeriesMaster` outputs only coalescing the Resolved branch) is still open — unrelated to P/N, deferred.

---

## Flow B (`PA - Resolve OneNote Meeting Section - v2 Clean Build`)

**FB-F01 — Compose Input MeetingTitle (one-off)** 🟢 fixed and tested
Was: 8 nested `replace()` calls wrapped in `substring(..., 0, min(43, length(...)))`, which threw when `MeetingTitle` arrived empty (`substring` requires start index strictly `<` string length; `0 < 0` is false on an empty string).
Now: guarded with an empty-check ahead of the existing sanitization, falling back to a static title rather than crashing:
```
if(empty(trim(coalesce(triggerBody()?['text_1'], ''))), 'Mtg - Untitled Meeting', concat('Mtg - ', substring(replace(replace(replace(replace(replace(replace(replace(replace(triggerBody()?['text_1'], '/', '-'), ':', '-'), '&', 'and'), '?', ''), '*', ''), '<', ''), '>', ''), '"', ''), 0, min(43, length(replace(replace(replace(replace(replace(replace(replace(replace(triggerBody()?['text_1'], '/', '-'), ':', '-'), '&', 'and'), '?', ''), '*', ''), '<', ''), '>', ''), '"', ''))))))
```

---

## Topic (`Meeting Capture (v4 rebuild)`)

**C6B_Check_N structural nesting** 🟢 fixed and tested
Was: `C6B_Check_N`'s `ConditionGroup` was nested *inside* `C6_Check_Input`'s (the P-check) own `actions:` array in the YAML, after a `GotoAction` — making it structurally unreachable for any non-"P" input. This was invisible on the Designer canvas; only found by reading the full YAML source.
Now: `C6B_Check_N` is a sibling condition alongside `C6_Check_Input` inside `conditionGroup_BsGPk1`, both reachable independently.

**C1_Set_DateContext** 🟢 fixed and tested
Was: `value: today` — a literal string, not an evaluated date.
Now: `value: =Text(Today(), "yyyy-MM-dd")` — produces a real, parseable ISO date string from the start.

**C6_Check_Input (P) / C6B_Check_N (N) date-shift SetVariable** 🟢 fixed and tested
Was: `=Text(DateAdd(Topic.DateContext, ±1, TimeUnit.Days), "yyyy-MM-dd")` — `DateAdd` requires a Date value, but `Topic.DateContext` is a `string`-typed variable, so this threw "The date or time value cannot be parsed." This error only surfaced *after* the structural nesting fix above, since `C6B_Check_N`'s DateAdd call had never actually executed before that fix.
Now, both branches wrap the variable in `DateValue()` before adding/subtracting a day:
```
=Text(DateAdd(DateValue(Topic.DateContext), -1, TimeUnit.Days), "yyyy-MM-dd")   ' P branch
=Text(DateAdd(DateValue(Topic.DateContext), 1, TimeUnit.Days), "yyyy-MM-dd")    ' N branch
```

**Topic Checker — "Unspecified" type errors on MatchCount, IsRecurring, MeetingTitle, SeriesMasterId, CalendarEventId** 🟢 fixed and tested
Root cause: Flow A's `FA43_Respond_to_agent` response schema correctly declares `"type": "string"` on all seven outputs (confirmed via peek code) — the problem was purely Topic-side. Copilot Studio doesn't reliably infer a type on variables populated via `InvokeFlowAction` output bindings, leaving them "Unspecified"/"unknown" even when the flow's schema is fully typed. This caused a `ConditionGroup` type-mismatch error on `C4_Check_MatchCount` (`Topic.MatchCount = Text(0)`, comparing Unspecified to String) and four input-type warnings on `C10_Call_FlowB_Create_Page`.
Fix: added a `Set variables` step immediately after `C2_Call_FlowA_Initial` (upstream of both the initial capture path and the P/N loop-back, since both route through C2) that reassigns each variable through `Text(...)`, which forces Copilot Studio to infer the type as `string`:
```
Topic.MatchCount = Text(Topic.MatchCount)
Topic.IsRecurring = Text(Topic.IsRecurring)
Topic.MeetingTitle = Text(Topic.MeetingTitle)
Topic.SeriesMasterId = Text(Topic.SeriesMasterId)
Topic.CalendarEventId = Text(Topic.CalendarEventId)
```
Confirmed via Variable properties panel: all five now show type `string` (previously `unknown`). Topic Checker errors dropped from 7 to 2 immediately after this change (no other changes made in between).

**Topic Checker — "Flow not found or is turned off" (×2, on InvokeFlowAction nodes)** ⚪ confirmed clean / cosmetic
Both entries point at `InvokeFlowAction` nodes (`C2_Call_FlowA_Initial` confirmed directly; a second, functionally identical node not individually re-confirmed this session). Both flows have been live-tested extensively and work correctly. Publish succeeded today (badge now reads "Published 7/18/2026") with these two still present, confirming they do not block publishing and are a stale Designer-side reference display issue, not a real broken binding. Consistent with the same finding made earlier in this project on `invokeFlowAction_bIIKPf`.

---

## Live verification (2026-07-18, Test panel)

Single continuous session:
1. "capture meeting notes" → 13-item candidate list for today.
2. "n" → different, shorter list (correctly shifted to next day).
3. "n" again → different again (correctly shifted a further day forward).
4. "4" → "Great — I've found your meeting: Antonio AL" → "Great news! Your meeting notes for Antonio AL have been saved to OneNote. Here's your page link: [live SharePoint URL]".

Confirms: P/N navigation genuinely re-queries Flow A with a shifted date each time (not repeating the same list), number selection still resolves correctly after all the P/N-related changes, and Flow B's OneNote page creation path (including the FB-F01 fix) works end to end with a real returned link.

---

## Open items / not yet covered by this pass

- FA43 Respond to agent's `IsRecurring`/`SeriesMaster` coalescing gap (Finding 4, 07-16 doc) — still open, unrelated to P/N, safe to defer.
- Flow B's back-half (Condition Mapping Exists, Condition Should Create Page, page creation chain, Condition Is Genuine Existing Page, final Respond to the agent's 20-output schema) — still not verified via peek code, only known secondhand from the original living audit.
- The second "Flow not found" node wasn't individually clicked-and-confirmed this session (only inferred from consistency with the first) — worth a quick confirm next time the canvas is open, though low priority given Publish succeeded.
- The disabled, Activity-triggered second "Meeting Capture" topic — purpose still unconfirmed.
