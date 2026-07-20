# Date-Jump Feature Build + Full UJ1–UJ5 Validation

**Date:** 2026-07-20
**Status:** Date-jump navigation built and validated. All five original user journeys (UJ1–UJ5) confirmed passing for the first time as a complete set.

---

## 1. Background

The candidate-list prompt has always advertised a third navigation option alongside P (previous day) and N (next day): "...or a date (e.g. 28 Jun) to jump." This was never actually implemented — the Topic only had explicit handling for "P"/"N"; any other input, including a typed date, was passed straight through into Flow A's number-selection field unchanged, which would have caused Flow A to attempt `int()` parsing on the date text and crash (the same class of bug originally found and fixed for P/N navigation on 2026-07-16/18).

This was discovered while testing UJ5 (no-match recovery) — attempting to navigate to a specific date to find an empty day surfaced that the feature didn't exist.

---

## 2. What was built

A new third condition, `C6C_Check_Date`, added as a sibling to `C6_Check_Input` (P) and `C6B_Check_N` inside `conditionGroup_BsGPk1` in the "Meeting Capture (v4 rebuild)" Topic.

**Condition expression:**
```
=IsError(Value(Topic.TopicSelectedNumber)) && !IsError(DateValue(Topic.TopicSelectedNumber))
```

This deliberately checks two things together: the input must **fail** to parse as a plain number (so genuine numeric selections like "3" are never intercepted) and must **succeed** parsing as a date. Ordering matters — since `conditionGroup_BsGPk1` evaluates in order and this condition sits after the P/N checks, "P" and "N" are still handled by their own conditions first.

**Actions inside the new condition:**
```yaml
- kind: SetVariable
  variable: Topic.DateContext
  value: =Text(DateValue(Topic.TopicSelectedNumber), "yyyy-MM-dd")

- kind: GotoAction
  actionId: invokeFlowAction_eBUGn8
```

This mirrors the exact pattern already used by P/N: compute a new `DateContext`, then loop back to `C2_Call_FlowA_Initial` to re-query Flow A for that date, rather than treating the text as a meeting selection.

---

## 3. Validation

### Format handling
- **Partial date, no year ("28 Jun")** — correctly resolved to `2026-06-28T00:00:00Z`, confirming `DateValue()` correctly assumes the current year for partial input.
- **Full date with explicit year ("10 Feb 2030")** — correctly resolved to `2030-02-10T00:00:00Z`, confirming explicit years are also handled correctly.

Both confirmed directly via Flow A's `FA06_Compose_StartOfDayUtc` output in the Activity trace, not just by absence of a crash.

### Interaction with number selection
No regression observed — plain numeric selections (tested repeatedly throughout the day) continued to route correctly into Flow A's number-selection path, confirming the `IsError(Value(...))` guard correctly excludes them from being misidentified as dates.

### Interaction with UJ5
Directly enabled UJ5's validation — jumping straight to a known/suspected empty date is far more reliable than paging through with repeated P/N presses. See `uj5-validation-record.md` for the full no-match test using this feature.

---

## 4. Known consideration, not yet an issue

Distant future dates are not guaranteed to be empty if a recurring meeting series has no end date — confirmed when "10 Mar 2027" (assumed to be a weekend, actually a Wednesday) returned four real candidates, several from standing recurring series. This is correct, expected behavior, not a bug — flagged here only so future testers don't mistake it for one.

---

## 5. Full UJ1–UJ5 status as of today

| Journey | Status | First passed |
|---|---|---|
| UJ1 — One-off single match | ✅ Pass | 2026-06-26 (re-confirmed 2026-07-20 after `FB-F01` change) |
| UJ2 — Multiple match selection | ✅ Pass | Informal since 2026-06-29; formally recorded 2026-07-20 |
| UJ3 — Recurring, existing mapping | ✅ Pass | 2026-07-19 (never passed before) |
| UJ4 — Recurring, first-time setup | ✅ Pass | 2026-07-19 (never passed before) |
| UJ5 — No-match recovery | ✅ Pass | 2026-07-20 (never tested before) |

This is the first point in the project's history where all five journeys have been independently confirmed working via live testing and Activity-trace evidence, rather than assumed or partially verified.

---

## 6. Open items / not yet covered

- `living-audit.md` still needs the individual per-action entries from this session and 2026-07-19 folded in (structural fixes, `Condition_IsRecurring`, section-creation chain, `Compose_SafeSectionName`, `OutStatus`, date-jump).
- FA43 `IsRecurring`/`SeriesMaster` coalescing gap (Flow A, Finding 4, 2026-07-16 doc) — still open, unrelated to today's work.
- Flow B's `OutStatus` still has no genuine error branch — today's fix corrected a regression back to `"OK"`, but the underlying architectural gap (no coalescing across failure branches, unlike Flow A) remains.
- Two different OneNote connections (`shared_onenote` vs `shared_onenote-1`) in Flow B — cosmetic inconsistency, not yet standardized.
