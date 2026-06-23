# Structural Fix Plan — 2026-06-21

This document supersedes pure bug-by-bug patching for the two highest-leverage, lowest-risk structural changes identified this session. Each fix below was stress-tested against a realistic future-failure scenario before being included — see `living-audit.md`'s open items and the session transcript for the reasoning. These are Designer-only changes; none require live Teams testing to verify correctness (verify in Power Automate's "Run flow" / Test pane instead), so they can be applied independent of the ongoing Flow B connectivity investigation.

**Sequencing recommendation:** apply these now, in parallel with — not blocked by — the Flow B "Flow not found or is turned off" connectivity investigation (see `living-audit-topic.md` Section 8), since that issue blocks live Teams testing but not Designer-level edits.

---

## Structural Fix 1 — Eliminate the parallel-array hazard (Flow A)

**Why this is structural, not a patch:** FA19's bug exists because two arrays carry "the candidates" — `FA09_Compose_CandidateArray` (raw, unfiltered) and `FA09A_Filter_CandidatesByTitle` (filtered, the one FA13's MatchCount and the user-facing candidate list are actually built from). Nothing currently prevents a future branch from referencing the wrong one, the same way FA19 did. Renaming the raw array makes the correct choice the obvious one for anyone building a new branch later, not just fixing today's instance.

### Step 1a — Fix FA19's source array (the confirmed bug)
**Action:** `FA19_Compose_SelectedEvent`
**Current:**
```
outputs('FA09_Compose_CandidateArray')[outputs('FA16_Compose_SelectedIndex')]
```
**Change to:**
```
outputs('FA09A_Filter_CandidatesByTitle')[outputs('FA16_Compose_SelectedIndex')]
```
**Why this is safe:** FA18's range check already validates the selected index against `FA13_Compose_MatchCount`, which is `length(body('FA09A_Filter_CandidatesByTitle'))` — the filtered array's length. This change makes FA19 consistent with the bound already being enforced upstream. No new failure mode.

**Verify after applying:** in Power Automate, use "Run flow" / Test with a UJ2-style scenario where the raw calendar return has more events than match the title filter (if no such test case exists naturally, this is itself worth creating as a standing test case). Confirm the selected event's title matches what the user actually picked from the displayed list.

### Step 1b — Rename the raw array to make misuse visible (the structural part)
**Action:** `FA09_Compose_CandidateArray`
**Rename to:** `FA09_RAW_CandidateArray_DoNotUseDownstream`
**Why this is safe:** Compose action display names don't affect execution — this is a labeling change only. Confirmed via full audit that only `FA09A`'s own `From` field references FA09 directly; no other action does. Renaming cannot break any existing binding.
**Why this matters:** the next time a branch is built that needs "the candidates," the array name itself signals which one to use, rather than relying on institutional memory of this specific incident.

---

## Structural Fix 2 — Two mechanical conventions, applied flow-wide

**Why this is structural, not a patch:** both conventions below address bug *categories* found in 4+ independent locations each. Fixing instances one at a time (as has happened across at least three prior sessions for the literal-`''` pattern specifically — FA33A/FA34A were fixed once already and found broken again) doesn't prevent recurrence. Adopting and documenting the convention does, because it becomes a single grep-able rule rather than per-instance judgment.

### Convention A: no bare `''` or blank Value/Inputs fields — always `string('')` for genuinely-empty outputs, or the real intended expression otherwise

**Mechanical check for future sessions:** in Code View, search for `"inputs": "''"` and any Set-variable/Compose action with an empty `"value"`/`"inputs"` field. Every match is a violation of this convention.

Fixes to apply now:

| Action | Current | Fix |
|---|---|---|
| `FA32_Compose_OutCandidateList_Single` | `''` | `string('')` |
| `FA23_Compose_OutCandidateList_Resolved` | `''` | `string('')` |
| Flow B `Compose_IgnoreSeriesMasterId` | `''` | `string('')` — **note:** confirm this is actually meant to always be empty; if it's meant to carry data this is a design question, not a mechanical fix |
| Flow B `varFinalExistingPageSelfUrl_1` | blank | restore real expression (not yet transcribed verbatim in `living-audit.md` — needs Designer lookup) |
| Flow B `varFinalPageDecision_1` | blank | restore real expression (same) |
| Flow B `varFinalMatchCount_1` | blank | restore real expression — **highest priority of this group**, this is the value that feeds `Condition_Should_Write_Mapping`'s crash |
| Flow B `Set varTargetSectionPagesUrl OneOff Exists` | blank | restore real expression |
| Flow B `Set varOneNoteResolverResult Exists OneOff` | blank | restore real expression |
| Flow B `Set varTargetSectionPagesUrl OneOff Created` | blank | restore real expression |
| Flow B `Set varOneNoteResolverResult Created OneOff` | blank | restore real expression |
| Flow B `Set varPageAction Created` | blank | restore real expression (likely literal `'Created'`) |
| Flow B `Set varOutputPageSelfUrl Created` | blank | restore real expression |
| Flow B `Set varOutputPageLink Created` | blank | `outputs('Create_Page_OneOff')?['body']?['links']?['oneNoteWebUrl']?['href']` — **confirmed correct expression from this morning's earlier fix; this is a regression, re-apply with confidence** |
| Flow B `Set varOutputPageLink Created OneOff` | blank | same expression as above |
| Flow B `Set varPageAction UpdatedAppend` | blank | restore real expression (likely literal `'UpdatedAppend'`) |
| Flow B `Set varOutputPageLink Existing` | blank | restore real expression |

### Convention B: numeric conversions must guard with `if(empty(...))`, never bare `coalesce()`

`coalesce(var, default)` only substitutes when `var` is **null** — it does nothing when `var` is an empty string, which is exactly the failure mode that crashed Flow B live. The correct pattern already exists once in this project, working correctly, thirty lines from the broken version:

**Working reference pattern** (`Condition Mapping Exists`, do not change, use as the template):
```
if(empty(coalesce(variables('varFinalMatchCount'), '')), '0', greater(int(coalesce(variables('varFinalMatchCount'), '0')), 0))
```

**Fix to apply** (`Condition_Should_Write_Mapping`, confirmed live-crash root cause):
Current:
```
greater(int(coalesce(variables('varFinalMatchCount'),'0')),0)
```
Change to:
```
greater(int(if(empty(variables('varFinalMatchCount')), '0', variables('varFinalMatchCount'))), 0)
```
This matches the already-proven working pattern rather than inventing a new one — lower risk, since the logic has already been validated elsewhere in the same flow.

**Also apply the same check to** `Condition Section Exists OneOff` (flagged as needing the same guard pattern, expression not yet fully expanded in `living-audit.md` — open this in Designer, confirm whether it has the same `coalesce`-without-`empty`-guard shape, and apply the same fix if so) and `Condition Should Create Page` (currently 🟡 unconfirmed/suspect, same family, not yet expanded).

### Convention B, related fix — missing `@` expression prefix

Same root cause (type-safety silently bypassed), different specific mechanism. Both confirmed via Designer screenshot this session (plain-text rendering instead of an expression chip):

**FA03_Init_varOriginalUserSearchText**
Current: `"value": "triggerBody()?['OriginalUserSearchText']"`
Fix: `"value": "@triggerBody()?['OriginalUserSearchText']"`

**FA04_Init_varDateContext**
Current: `"value": "triggerBody()?['DateContext']"`
Fix: `"value": "@triggerBody()?['DateContext']"`

**Mechanical check for future sessions:** any Set-variable action whose Value renders as plain unstyled text in Designer (rather than a colored expression chip) is missing its `@` prefix.

---

## What this plan deliberately does NOT include (sequencing decision, not oversight)

**Consolidating the `Respond to the agent` field construction into a single object-build step** (the fix for FA43's missing-branch bug class) is a real structural improvement but was deliberately held back from this immediate pass — it requires restructuring FA19-26 and Flow B's equivalent response-building chain, and should be verified end-to-end via live Teams testing once built, which is currently blocked by the Flow B connectivity issue. Apply Structural Fixes 1 and 2 first; revisit this once Flow B connectivity is restored and FA19/FA43's individual fixes (drafted separately, see `living-audit.md`) are confirmed working live.

**The `runAfter` casing question** (`"SUCCEEDED"` vs `"Succeeded"`) remains deliberately unresolved — Flow A uses the same casing as Flow B throughout, which is evidence against this being a bug at all. Do not "fix" this until definitively confirmed against Power Automate's actual requirement.

**`FA12`'s IsRecurring derivation** and **`FA09A`'s fallback-source risk** remain open — both need a downstream-consumption confirmation before it's worth prioritizing a fix, per the existing open items in `living-audit.md`.

---

## Verification checklist (Designer-only, no live Teams testing required)

After applying Structural Fixes 1 and 2:
1. Flow A and Flow B Flow Checker — confirm 0 operation errors (previously seen 15 errors of exactly this `'Value' is required` shape; this checklist should clear all of them).
2. Power Automate "Test" / "Run flow" — manually trigger Flow B with the QWE Meeting values used in the 2026-06-20 evening session, confirm `Condition_Should_Write_Mapping` no longer throws `int()` InvalidTemplate.
3. Power Automate "Test" / "Run flow" — manually trigger Flow A with a UJ2-style multi-match scenario, confirm FA19 selects the event matching the user's actual numbered choice from the candidate list, not a different one from the unfiltered array.
4. Update `living-audit.md` to flip each fixed action's status from 🔴 to 🟢, with a one-line "confirmed via manual Test run, 2026-06-XX" note per this project's existing convention.
5. Only once 1-4 pass cleanly: proceed to the Flow B connectivity investigation (`living-audit-topic.md` Section 8) and, once resolved, live Teams re-test.
