# Handover Prompt — Next Session

**Project:** Teams → OneNote Meeting Capture
**GitHub:** Crox2310/Teams-OneNote-Meeting-Capture
**Date of last session:** 26 September 2026

---

## What happened this session

Flow A v2 (`PA - Resolve Meeting Selection - v2`, flowId `12b1d0c8-a6b9-f111-aaaf-7ced8d745465`) was built from scratch using the new Compose-based, Scope-organised methodology, published, tested directly, and switched live in the agent. The Topic (`Meeting Capture (v4 rebuild)`) now calls v2. End-to-end confirmed in Teams: candidate list displayed correctly, numbered 1-3, correct date header.

## Current state

**Flow A v2:** Published, live, working.
**Flow B:** Still v1, untouched this session. Known corruption issues.
**Flow C:** Published and live (AI insight capture + chat fallback).
**Flow D:** Published and live (auto-capture scheduler).
**Topic:** Updated to call Flow A v2 via `invokeFlowAction_rLKsKC`.

## Immediate blockers

### 1. Number selection broken (HIGH — blocks all multi-match captures)
C6D was removed from the Topic because `Topic.CandidatesJson` variable no longer exists (v2 doesn't output `candidatesjson`). Currently if a user types a number to select from a multi-match list, the agent says "That number isn't in the list."

**Fix:** Add a new Compose action in v2's `Scope_Response` that builds a JSON array of candidates and outputs it as `candidatesjson`. Then restore C6D to the Topic YAML.

The JSON structure needed for each candidate:
```json
{"Title": "meeting subject", "Id": "calendarEventId", "IsRecurring": "true/false", "SeriesMasterId": "...", "BodyPreview": "...", "OnlineMeetingUrl": "...", "End": "..."}
```

This Compose sits in `Scope_MultiMatch` after MM04 (the loop builds the candidate list — the JSON needs to be built in the same loop, appending each item). Or more precisely: a new `Scope_Response` Compose that reads back from `CA07_Compose_Sorted_Candidates` and builds the JSON array using `string()` + a Select or concat approach.

### 2. Flow A v1 still connected in Connection Settings
Connection Settings shows v1 as the live connection, not v2. This hasn't caused a runtime problem (v2 is working) but is untidy. Low priority.

### 3. TV02 value wipes on Designer open
Same corruption pattern as v1 FA34A. At the start of every session: open v2, wait 20s, Peek Code TV02, confirm value = 1. Restore if blank before doing anything else.

## Next session priority order

1. **Fix `candidatesjson`** — add it to v2's Scope_Response output so number selection works
2. **Run a full end-to-end capture** — confirm the full journey (list → select → OneNote page created) works on v2
3. **Apply BL-36 fix** — duplicate sections bug: gate `Condition_Section_Count_Is_Zero` on `IsRecurring` in Flow B
4. **Flow B rebuild** — see below

## Flow B rebuild consideration

Flow B (`PA - Resolve OneNote Section`) is the next major rebuild candidate. It has a severe and recurring corruption problem (SetVariable/InitializeVariable value wipes on Designer open, affecting 15-33 actions simultaneously). The v2 methodology (Compose-based state passing, no SetVariable except top-level InitializeVariable) would eliminate this class of corruption entirely.

Flow B is significantly more complex than Flow A — it handles:
- Recurring vs one-off meeting routing
- SharePoint mapping lookup (existing row vs new row)
- OneNote section resolution (existing section, create section, D2 fallback)
- Page creation vs page append
- SharePoint write-back (OccurrenceDate, JoinUrl, EndTime, PageWebUrl, PageSelfUrl, RecapCaptured)
- 6 output status values (SUCCESS/PARTIAL_SUCCESS/STALE_MAPPING/RECURRING_SETUP_REQUIRED/SETUP_SECTION_NOT_FOUND/SETUP_SECTION_AMBIGUOUS)

Estimated rebuild: 3-4 sessions. Same methodology as Flow A v2:
- Lock trigger contract and output contract before building
- Scope-organised structure
- Compose-based state passing throughout
- No SetVariable except unavoidable top-level InitializeVariable actions
- GitHub push after each Scope confirms green

Before starting Flow B rebuild, use Opus to do the design/architecture pass (trigger contract, output contract, Scope structure, branch logic) — same as the design work done for Flow A. This is worth a dedicated session or at least the first half of the next session.

## Key file locations on GitHub

```
flows/
  BUILD-METHODOLOGY.md          — core principles, platform constraints, health check protocol
  BUILD-PROMPT.md               — master build prompt for session start
  flow-a-v2/
    trigger-contract.md         — text=InSelectedNumber, text_1=DateContext
    output-contract.md          — 10 output fields
    known-good-values.md        — all expressions, TV02 wipe warning
    EXPRESSION-REFERENCE.md     — trigger key corrections
    TOPIC-SWITCHOVER-STATUS.md  — candidatesjson gap documented
    scope-peek-codes/
      scope-flow-a.md           — full verified Scope Peek Code (26 Sep 2026)
```

## Platform constraints confirmed during v2 build

| Constraint | Detail |
|---|---|
| `indexOf()` array | NOT supported — use varCandidateIndex counter |
| `InitializeVariable` nesting | Top-level only — not inside Scope or Condition |
| Dynamic Content picker | Inserts internal IDs not action names — always type manually |
| Trigger key assignment | New Designer assigns sequentially: `text`, `text_1`, `text_2`... |
| `filter()` in Compose | Not available — use Filter Array (Query) action |
| TV02 value wipe | Wipes on Designer open — check at session start |

## Flow A v2 known-good values (check at session start)

| Ref | Action | Value |
|---|---|---|
| TV01 | TV01 Initialise Candidate List Text | (blank string) |
| TV02 | TV02 Initialise Candidate Index | `1` (integer) — WIPES ON OPEN |
| CA01 | CA01 Compose DateContext | `@coalesce(triggerBody()?['text_1'], utcNow())` |
| CR02 | CR02 Compose Display Date | `@formatDateTime(concat(outputs('CA01_Compose_DateContext'), 'T12:00:00'), 'ddd d MMM yyyy')` |
