# Bug — FA16_Compose_SelectedIndex crashes on date text reaching Flow A unguarded (20 August 2026)

## Status
**Confirmed via live run trace. Root cause partially identified — defensive fix is clear, but why the date text reached Flow A at all still needs investigation.**

## Origin
Raised by David as field observation #3 on 20 August 2026: date entry at the "To change day select letter or enter date" prompt required an exact format (`20/08/26` worked; `20 Aug`/`20 aug` reportedly did not).

## What we actually found

The premise of the original report doesn't hold up under testing: `20 Aug` and `20 aug` both parsed correctly and returned the identical, correct candidate list for that date — same result as the known-working `20/08/26` baseline. `DateValue()` (used by the Topic's `C6C_Check_Date`, built 20 July) is not case-sensitive and handles `D MMM` format without a year correctly, consistent with its original validation on 20 July.

**However**, a live `FlowActionBadGateway` error occurred during this same test session — on the *third* date entry in the sequence (`20/08/26`, David's known-good baseline format), not on either of the two "new" formats tested first. Since the same input that worked earlier in the day failed here, this ruled out date-format parsing as the cause and pointed at something session/sequence-dependent instead.

## Root cause (confirmed via Activity trace)

The actual failing action, per the Flow A run trace (`PA - Resolve Meeting Selection - v1 Clean Build`, run at 8/20/26 10:12 PM):

> `FA16_Compose_SelectedIndex` failed: *"The template language function 'int' was invoked with a parameter that is not valid. The value cannot be converted to the target type."*

`FA16`'s expression:
```
@if(equals(trim(variables('varInSelectedNumber')), ''), 0, sub(int(trim(variables('varInSelectedNumber'))), 1))
```

This calls `int()` unconditionally on `varInSelectedNumber` whenever it's non-empty. **It has no exclusion for date-like text.** This is the same bug class as `FA15_Compose_IsSelectionMode`'s original bug (fixed 18 July) — that fix excluded `'NONE'`, `'P'`, and `'N'` from being force-parsed as a number:
```
and(not(empty(trim(variables('varInSelectedNumber')))), not(contains(createArray('NONE','P','N'), toUpper(trim(variables('varInSelectedNumber'))))))
```
But **dates were never added to that exclusion list on either `FA15` or `FA16`**. Whenever a typed date reaches Flow A's number-selection path — rather than being caught upstream by the Topic's `C6C_Check_Date` condition — this crash is inevitable, on any date format, not a specific one.

## Open question — why did the date reach Flow A at all?

The Topic's `C6C_Check_Date` (built 20 July, in `conditionGroup_BsGPk1`) is specifically designed to intercept date-like input *before* it reaches Flow A, looping back through `C2_Call_FlowA_Initial` with a shifted `DateContext` rather than passing the raw text into the selection field. Two prior date entries in this same session (`20 Aug`, `20 aug`) were correctly intercepted this way — confirmed by both returning shifted candidate lists rather than errors. The third entry (`20/08/26`) was not.

This is not yet understood and needs its own investigation, ideally following the same evidence-first approach that found the original P/N nesting bug (18 July) — that bug was invisible on the Designer canvas and only found by reading the full Topic YAML. Plausible hypotheses, **none yet confirmed**:
- `conditionGroup_BsGPk1` may not re-evaluate identically after a `GotoAction` loop-back, similar in spirit to the original P/N nesting issue.
- Something about three consecutive date-jump loops specifically (rather than two) may expose a state/variable issue — worth checking whether `Topic.TopicSelectedNumber` or `Topic.DateContext` is being read stale after repeated loops.
- Slash-format dates (`20/08/26`) specifically might behave differently in the `IsError(Value(...))`/`IsError(DateValue(...))` guard than space-format dates (`20 Aug`) — worth testing in isolation, since `Value()` (not `DateValue()`) is the function used in `C6C`'s numeric-exclusion check, and its behaviour on slash-delimited text hasn't been verified.

## Recommended fix

Two separate pieces of work:

1. **Defensive fix in Flow A (clear, low-risk)**: apply the same exclusion pattern to `FA16_Compose_SelectedIndex` that `FA15` already has, so that even if date-like text does leak through, the flow fails safely (e.g. resolves to a no-op or a clear error) rather than crashing with a raw `int()` template error. This won't fix the underlying routing gap, but stops it from causing a hard failure.
2. **Topic-routing investigation (needs its own session)**: read the full `conditionGroup_BsGPk1` YAML (not just the Designer canvas) to understand why the third date entry in this session bypassed `C6C_Check_Date`'s interception. Do not guess at a fix without first reproducing and inspecting the actual YAML, consistent with this project's established evidence-first working method.

## Related items

- Confirms `C6C_Check_Date` itself (20 July) is sound for the cases it does catch — this is a routing/interception gap, not a parsing regression.
- Same underlying bug class as the original `FA15` fix (18 July) — worth a broader sweep of Flow A for any other `int()`/`Value()` calls that assume purely numeric input without excluding known non-numeric control values (`P`, `N`, dates).

---
*Confirmed 20 August 2026 via live Teams session (three consecutive date-jump attempts) and Flow A Activity trace for the failing run (8/20/26 10:12 PM). Peek Code for `FA16_Compose_SelectedIndex` captured directly from the failed run.*
