# Addendum: Confirmed Root Cause — One-Off Existing-Page BadRequest

**Date:** 29 July 2026, ~23:40
**Parent document:** `handover-2026-07-29-oneoff-badrequest-investigation.md`
**Status:** Root cause confirmed via Peek Code + Activity trace. Fix not yet implemented.

---

## Summary

The original handover identified that `varFinalExistingPageSelfUrl` is initialized to `null` and never Set anywhere in the one-off execution path. This session traced that gap forward through Flow B (`PA - Resolve OneNote Meeting Section`) and confirmed exactly how it manifests as a runtime failure, via Peek Code on every action in the dependency chain plus one live Activity trace showing the failure occur in production.

**This is a single confirmed root cause, not multiple defects.** A hypothesis raised mid-session (a second instance of the "missing `value` key" pattern, logged previously as pattern 6 / AMEND-2026-07-27-002) was tested directly against Peek Code and **ruled out** — worth recording so it isn't re-checked in a future session.

---

## Confirmed causal chain

Flow B's `Condition Should Create Page` → **False** branch (page exists, don't create) runs:

```
Set varPageAction ExistsNoCreate
  → Set varOutputPageSelfUrl Existing
    → Compose UpdateHtmlFragment
      → Compose ExistingPageId
        → Condition Is Genuine Existing Page
```

**Step-by-step failure:**

1. **One-off meetings never populate `varFinalExistingPageSelfUrl`.** Confirmed independently two ways this session:
   - Peek Code shows no SetVariable action targeting it anywhere in the one-off branch of `PA - Resolve OneNote Meeting Section`.
   - A live Activity trace (10:10:58 PM run) shows the entire `varFinal*` chain — including `varFinalExistingPageSelfUrl` — sitting on the `IsRecurring = True` side of a condition, and marked `Skipped` (`ActionConditionFailed`) on the one-off run. It stays at its initialized value: `null`.

2. **`Set varOutputPageSelfUrl Existing` copies that null straight across.** Peek Code:
   ```json
   "name": "varOutputPageSelfUrl",
   "value": "@variables('varFinalExistingPageSelfUrl')"
   ```
   The `value` key **is present and correctly wired** — this is not a pattern-6 defect. It faithfully copies whatever `varFinalExistingPageSelfUrl` currently holds, which for one-off meetings is always `null`.

3. **`Compose ExistingPageId` operates on that null value:**
   ```
   last(split(variables('varOutputPageSelfUrl'), '/'))
   ```
   `split()` on a null/empty string is the kind of expression that throws at runtime in Power Automate rather than degrading gracefully to an empty result.

4. **The failure cascades to `Condition Is Genuine Existing Page`**, which depends on `Compose_ExistingPageId` via `runAfter: Succeeded`. A live Activity trace (23:11 PM run) shows this condition failing with:
   > `ActionFailed. An action failed. No dependent actions succeeded.`

5. **Downstream effect:** the True branch of `Condition Is Genuine Existing Page` (update existing page) can never be reliably reached for one-off meetings, because the action that gates it errors out first. The practical risk this creates — silent duplicate page creation on recapture — is the same risk described in the original AMEND-2026-07-27-001 finding, but this trace shows it is not fully resolved: the underlying data gap (`varFinalExistingPageSelfUrl` never populated for one-off) still allows the failure path to be hit.

---

## Hypothesis ruled out this session

**Pattern-6 (missing `value` key on SetVariable), second instance:** Before pulling Peek Code, it was reasonable to suspect `Set varOutputPageSelfUrl Existing` had the same "name only, no value" defect found and fixed elsewhere on 27 July. Peek Code shows this is **not the case** — the value assignment is correct. The defect is upstream (the source variable is never set), not in this action itself. No fix needed on `Set varOutputPageSelfUrl Existing`.

---

## What's still unknown / next steps

The original open question stands and is now sharper: **for a one-off meeting, where should the existing page's self-URL actually come from?**

Candidate source, not yet confirmed: within `Condition Is Genuine Existing Page`'s own True branch, `Get Sections Existing Branch` → `Filter Existing Section By Name` retrieves section data that plausibly contains the existing page reference needed — but this branch runs *after* the point where `varOutputPageSelfUrl`/`Compose ExistingPageId` already need a value, so the sequencing needs checking, not just the data source.

**Recommended next steps, in order:**

1. Determine whether a one-off equivalent to `Get Sections Existing Branch` (or its output) already resolves an existing page URL earlier in the flow than `Compose ExistingPageId` — if so, wire `varFinalExistingPageSelfUrl` (or feed `varOutputPageSelfUrl` directly) from that source for the one-off path.
2. If no such earlier resolution exists, a new one-off page-lookup step is needed before `Compose ExistingPageId` runs — mirroring what the recurring branch does via its `varFinal*` chain, but scoped to one-off meetings.
3. Live-verify the fix by re-running a one-off meeting against a section that already contains a matching page, and confirm `Condition Is Genuine Existing Page` evaluates to True (or False) without an `ActionFailed`.
4. Log the fix as a new amendment once verified, referencing this addendum and the original 2026-07-27 handover.
5. Backfill amendment-log.md per the outstanding 20 July gap-analysis item — this fix should be logged there too, not just in session handover docs.

---

## Evidence trail (for reference)

- Activity trace, 10:10:58 PM run — shows one-off branch taking `IsRecurring = False`, `varFinal*` chain skipped.
- Activity trace, 23:11 PM run — shows `Condition Is Genuine Existing Page` → `ActionFailed`.
- Peek Code, `Condition Is Genuine Existing Page` — full expression and both branches' actions.
- Peek Code, `Compose ExistingPageId` — `last(split(variables('varOutputPageSelfUrl'), '/'))`.
- Peek Code, `Set varOutputPageSelfUrl Existing` — confirms `value` correctly set from `varFinalExistingPageSelfUrl`, ruling out pattern-6.
