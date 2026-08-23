# Session note — 23 August 2026, part 3 (FR-01)

**Context:** Continuation of the same-day session, following BUG-01 resolution (part 1) and FR-03/FR-02/BUG-02 (part 2).

**Model/effort:** Sonnet 4.6, Standard throughout.

---

## FR-01 — Chronological candidate list ordering

**Investigation:** pulled a live Activity trace raw output from `FA08_Get_calendar_view_of_events` on a real multi-meeting day and compared Graph's return order against start times directly. Confirmed the original scoping note's caveat was correct: **Graph does NOT return events in chronological order.** Example from the live trace — item 3 in the raw array ("Logistics Product Weekly Team Call", 07:30) appeared after items 1 and 2 (both later, 09:00 and 08:30). This confirmed FR-01 was a genuine bug requiring a fix, not a non-issue.

Secondary confirmation: FR-02's filter (built earlier the same session) correctly excluded all 5 non-meeting entries present in the same raw payload (Sarra Beattie A/L, P7 W2 period reminder, Oraena A/L, Joey OOO, Quiet Hour), validating that fix continues to hold on a real, richer day.

**Design:** inserted a new Compose action `FA09C_Sort_CandidatesByStartTime` immediately after `FA09B_Filter_ExcludeLeaveAndPeriodEntries`, sorting the filtered array by `start` ascending, then repointed the same 6 downstream consumers used in FR-02's repoint pattern (`FA11`, `FA13`, `FA28`, `FA19`, `FA35`) from `FA09B` to `FA09C`.

**One real bug found and fixed safely, before touching production, using the `PA - Scratch Diagnostics` flow:**

WDL's `sort()` function does not accept the lambda/arrow-function key-selector syntax (`(item) => item['start']`) — this is the same class of error as the `isMatch()` mistake earlier in the day (assuming JS/Power-Fx-style syntax where WDL has its own). First attempt in the scratch flow failed with "The expression is invalid" at save time. Second attempt using `item()?['start']` as the key argument failed at runtime with a clear, informative error: `The template language function 'sort' expects its second parameter to be of type string. The provided value is of type 'Null'.` This directly revealed the correct signature — **`sort(array, 'propertyName')`, where the second parameter is a plain string field name, not an expression** — confirmed working on a third attempt with a hardcoded test array (sorted A/B/C correctly by `start`) before being applied to Flow A.

This is the second time in one day the scratch-flow pattern caught a syntax error before it could touch a production flow (the first being implicit — the `isMatch` error was caught live rather than in scratch, since FR-02 didn't use the scratch flow for that check). **Recommend continuing to default to `PA - Scratch Diagnostics` for any expression whose exact syntax isn't already proven in this codebase**, rather than testing new WDL function syntax directly against Flow A/B.

**Final confirmed expression (`FA09C_Sort_CandidatesByStartTime`):**
```
@sort(body('FA09B_Filter_ExcludeLeaveAndPeriodEntries'), 'start')
```

**Six downstream consumers repointed and verified via Peek Code:**
- `FA11_Apply_to_each_Candidates` → `@outputs('FA09C_Sort_CandidatesByStartTime')`
- `FA13_Compose_MatchCount` → `@length(outputs('FA09C_Sort_CandidatesByStartTime'))`
- `FA28_Compose_SingleEvent` → `@outputs('FA09C_Sort_CandidatesByStartTime')[0]`
- `FA19_Compose_SelectedEvent` → `@outputs('FA09C_Sort_CandidatesByStartTime')[outputs('FA16_Compose_SelectedIndex')]`
- `FA35_Apply_to_each_CandidateArray_ForList` → `@outputs('FA09C_Sort_CandidatesByStartTime')`

Note: `FA09C` is a Compose action, so its output is read via `outputs(...)` — unlike `FA09B` (a Filter array action, read via `body(...)`). This distinction was correctly applied throughout the repoint.

Published, Flow Checker 0 errors. **Confirmed working live** — candidate list now displays in correct chronological start-time order.

---

## Status at end of session (full day)

All items raised across the day are now resolved:

| Item | Status |
|---|---|
| BUG-01 — Second-occurrence recurring capture overwrite | ✅ Resolved |
| Flow A corruption (first-ever incident) | ✅ Fixed |
| FR-03 — OneNote link shortening (hyperlink approach) | ✅ Live |
| FR-02 — Holiday/leave/period/admin-block filter (11 patterns) | ✅ Live |
| BUG-02 — Zero-match day navigation gap | ✅ Resolved |
| FR-01 — Chronological candidate list ordering | ✅ Live |

**Genuinely all "interesting" items from the 22 Aug backlog and today's live discoveries are now closed.**

## What remains — explicitly lower priority, not urgent

| Item | Notes |
|---|---|
| UJ3b — Automatic stale-row cleanup | Resilience/edge-case hardening only, not fixing anything currently broken. |
| UJ4a — Section choice disambiguation | No design work started. Same category as above. |
| UJ4c — SectionRetryCount retry loop | Higher corruption risk (Do Until shape) than the other two. |
| `Condition_Should_Write_Mapping` explicit guard | Defense-in-depth, root cause already fixed upstream. |
| Flow A solution-aware / VS Code editable | One-time setup step. |
| **Microsoft support ticket** | **Still not submitted.** Now has 12+ documented incidents across 3 flows, plus today's evidence of a first-ever Flow A hit. This is the one item that keeps being deferred despite mounting justification. |
| Amendment log | Needs the full 23 August change set added (BUG-01, Flow A corruption, FR-03, FR-02, BUG-02, FR-01). |

---
*Written 23 August 2026.*
