# Flow A (PA - Resolve Meeting Selection) — P/N Day-Navigation Root Cause Diagnosis

**Date:** 2026-07-16
**Method:** Full peek-code trace of Flow A (Copilot Studio Designer → Code view, action by action) cross-checked against a live failed run (Client tracking ID `08584187523006635099393798256CU10`, 2026-06-30 10:23 PM).
**Status:** Root cause confirmed with direct evidence. Two Flow A fixes drafted below. One additional gap identified that lives outside Flow A (Copilot Studio Topic), not yet fixed.

---

## 1. Background

Phase 2 validation (see `12-phase-2-validation/handover-2026-06-29.md`) flagged that UJ2 number-selection passes, but P/N (previous/next day) navigation still crashes at `FA16_Compose_SelectedIndex` with:

```
Unable to process template language expressions in action 'FA16_Compose_SelectedIndex'
inputs at line '0' and column '0': 'The template language function
'int' was invoked with a parameter that is not valid. The value cannot be converted to the target type.'
```

The handover's leading theory was a whitespace/sentinel mismatch (the flow expecting a literal `"NONE"` sentinel, but the Topic possibly still sending a blank space for P/N calls). This document supersedes that theory with a confirmed, evidence-based root cause.

To investigate, Flow A was traced end-to-end via Copilot Studio's Code view (peek code) for every action from the trigger through to `FA43 Respond to agent`, then cross-checked against the actual inputs/outputs of a real failed run.

---

## 2. Confirmed findings

### Finding 1 — The IsSelectionMode gate only recognizes `NONE`, not `P`/`N`

`FA15_Compose_IsSelectionMode`:
```
@and(
  not(empty(trim(variables('varInSelectedNumber')))),
  not(equals(toUpper(trim(variables('varInSelectedNumber'))), 'NONE'))
)
```

This correctly excludes an empty value and the literal string `"NONE"` from being treated as a numeric selection. It has **no exclusion for `"P"` or `"N"`**. Anything else non-empty is assumed to be a candidate number and is passed to:

`FA16_Compose_SelectedIndex`:
```
@if(equals(trim(variables('varInSelectedNumber')), ''), 0, sub(int(trim(variables('varInSelectedNumber'))), 1))
```

If `varInSelectedNumber` is `"N"` or `"P"`, `int(trim('N'))` throws — this is the exact error text seen in production.

### Finding 2 — Live run evidence confirms the exact failure mechanism

Run `08584187523006635099393798256CU10` (2026-06-30, 10:23:05 PM), which failed at `FA16_Compose_SelectedIndex` with the error above:

- `FA02_Init_varInSelectedNumber` → `"value": "N"` (the literal letter, not a space, not "NONE")
- `FA04_Init_varDateContext` → `"value": null` (never populated by the Topic at all)

This directly confirms Finding 1's mechanism: the Topic sends the raw direction letter `"N"` into the `InSelectedNumber`/`text_1` trigger parameter — the same parameter used for numeric meeting selection — and Flow A has no case for it.

### Finding 3 — FA06/FA07 never consume `varDateContext`

`FA06_Compose_StartOfDayUtc`:
```
@formatDateTime(utcNow(),'yyyy-MM-ddT00:00:00Z')
```
`FA07_Compose_EndOfDayUtc`:
```
@formatDateTime(utcNow(),'yyyy-MM-ddT23:59:59Z')
```

Both are built purely from `utcNow()`. `varDateContext` (set from trigger field `DateContext` at `FA04`) is never read again anywhere downstream. `FA08_Get_calendar_view_of_events` uses `FA06`/`FA07`'s outputs directly, so the calendar query window is always "today," regardless of any day-paging intent.

Combined with Finding 2 (`DateContext` was `null` on the failed run), this confirms there is **no date-offset computation happening anywhere in this system today** — not in Flow A, and not in the Topic either. The Topic is not silently doing the math and failing to pass it through; it simply isn't doing the math at all yet.

### Finding 4 — Secondary, lower-priority defect: FA43 Respond to agent output coalescing is inconsistent

Of the seven fields returned by `FA43 Respond to agent`, five (`Status`, `MatchCount`, `CandidateList`, `MeetingTitle`, `CalendarEventId`) are coalesced across every branch (NoMatch → Single → Multi → Error → fallback). Two are not:

- `IsRecurring` → wired to `outputs('FA19B_Compose_OutIsRecurring_Resolved')` only
- `SeriesMaster` → wired to `outputs('FA19C_Compose_OutSeriesMasterId_Resolved')` only

Both only populate on the number-selection ("Resolved") path. Equivalent composes exist for the other branches (`FA28A`/`FA28B` for Single, `FA27G`/`FA27H` for NoMatch, `FA43A`/`FA43B` for Multi) but aren't wired into the final response. Any non-Resolved-branch response will return blank/null for these two fields. This is unrelated to the P/N crash and can be fixed independently.

---

## 3. Root cause statement

P/N day-navigation was never fully implemented end-to-end. The Topic sends a direction indicator (`"P"`/`"N"`) into the wrong parameter (`InSelectedNumber`, meant for numeric candidate selection) and does not compute or send a target date via `DateContext` at all. Flow A, in turn, has no logic to recognize `P`/`N` as anything other than an invalid number, and even if it did, has no mechanism to turn a direction into an actual query date since `DateContext` never arrives populated. The `int()` crash is a symptom of both gaps meeting at `FA16`.

---

## 4. Recommended fixes — Flow A (safe to deploy now)

These stop the crash and make Flow A capable of honoring a date once the Topic starts sending one. They do **not**, by themselves, make P/N paging work (see Section 5).

### Fix 1 — `FA15_Compose_IsSelectionMode`: exclude P/N from numeric-selection handling

```json
{
  "type": "Compose",
  "inputs": "@and(not(empty(trim(variables('varInSelectedNumber')))), not(contains(createArray('NONE','P','N'), toUpper(trim(variables('varInSelectedNumber'))))))",
  "runAfter": {
    "FA14_Compose_CandidateList": [
      "SUCCEEDED"
    ]
  }
}
```

### Fix 2 — `FA06`/`FA07`: consume `varDateContext` with a same-day fallback

`FA06_Compose_StartOfDayUtc`:
```json
{
  "type": "Compose",
  "inputs": "@formatDateTime(if(empty(trim(coalesce(variables('varDateContext'), ''))), utcNow(), variables('varDateContext')), 'yyyy-MM-ddT00:00:00Z')",
  "runAfter": {
    "FA03A_DEBUG_RawInputs": [
      "SUCCEEDED"
    ]
  }
}
```

`FA07_Compose_EndOfDayUtc`: identical pattern, format string `'yyyy-MM-ddT23:59:59Z'`, `runAfter: FA06_Compose_StartOfDayUtc: SUCCEEDED`.

### Fix 3 (optional, independent) — FA43 coalescing for IsRecurring/SeriesMaster

Wrap both fields in the same coalesce pattern already used for the other five fields, pulling from `FA28A`/`FA19B`/`FA27G`/`FA43A` (IsRecurring) and `FA28B`/`FA19C`/`FA27H`/`FA43B` (SeriesMaster) in branch order.

---

## 5. Remaining gap — this is Topic work, not Flow A work

Deploying the fixes above stops the crash but will **not** make "next day"/"previous day" actually change what's shown, because nothing computes a new date. The Copilot Studio Topic needs to:

1. Track the currently-displayed date as a Topic variable (it must already render something like "Monday, July 14" to the user, so this state likely exists in some form).
2. On a P/N button press, compute the new date (`DateAdd(currentDate, 1, Days)` or `-1`) and re-invoke Flow A passing that computed date as `DateContext`.
3. Stop sending `"P"`/`"N"` into `InSelectedNumber` — that parameter should stay `"NONE"` for a day-navigation call, since it isn't a meeting selection.

Once wired, Fix 2's fallback picks up the Topic-supplied date automatically — no further Flow A change needed.

---

## 6. Suggested next steps

1. Deploy Fix 1 and Fix 2 to Flow A (safe, non-regressive, stops the live crash).
2. Add the Topic-side date-tracking/computation work described in Section 5 to the backlog — this is the actual remaining blocker for P/N to function, not a Flow A defect.
3. Once both are live, run one live P/N test and confirm the returned meetings shift by a day. If they don't, the Topic-side wiring is the next thing to check (confirm `DateContext` is populated on the call, per the method in Section 2).
4. Fix 3 (FA43 coalescing) can be picked up independently, unrelated to P/N.
5. Resume UJ3–UJ5 validation once P/N is confirmed working end-to-end, per `phase-2-validation-plan.md`'s sequential rule.

---

## Appendix — Evidence trail

- Failed run: Client tracking ID `08584187523006635099393798256CU10`, 2026-06-30 10:23:05 PM (local).
- `FA02_Init_varInSelectedNumber` output: `"value": "N"`.
- `FA04_Init_varDateContext` output: `"value": null`.
- `FA06_Compose_StartOfDayUtc` output on this run: `"2026-06-30T00:00:00Z"` (today's date, not a shifted date).
- `FA07_Compose_EndOfDayUtc` output on this run: `"2026-06-30T23:59:59Z"`.
- Failure point: `FA16_Compose_SelectedIndex`, `InvalidTemplate` / `int` conversion error.
