# Microsoft Power Automate Platform Corruption — Discussion Brief

**Prepared:** 22 August 2026
**Purpose:** briefing document for direct discussion with Microsoft representative

---

## Environment details

- **Environment ID:** `76f9c3bd-16c5-e540-8bb4-7171f4745b45`
- **Platform:** Copilot Studio (Power Automate flows embedded)
- **Tenant:** Sainsbury's (`e11fd634-26b5-47f4-8b8c-908e466e9bdf`)
- **Primary flow affected:** `PA - Resolve OneNote Meeting Section - v2 Clean Build` (`ed112c88-b94b-f111-bec6-002248a38052`)
- **Secondary flow affected:** `Copilot-Email-Triage-Agent` (separate repo, same environment, same pattern observed)

---

## The bug

`SetVariable` and `Compose` actions silently lose their `value` / `inputs` field. The action definition remains structurally valid — it retains the correct `name`, `type`, `runAfter`, and `metadata` — but the expression or literal value is completely absent.

Flow Checker flags these as `'Value' is required` but only detects approximately 60–70% of affected actions. Some blanked values pass Flow Checker silently, making the corruption harder to catch and recover from.

---

## Why this is particularly damaging

- **Bulk impact:** typically 20–26 actions are affected simultaneously, across unrelated parts of the flow, in a single incident
- **Bystander actions:** actions that were not edited in the session lose their values when neighbouring actions are changed
- **No undo:** there is no way to recover the previous value from within the Designer; the only recovery path is an external known-good reference document maintained manually
- **Silent failures:** a flow with blanked values will often still pass Flow Checker and publish successfully, then fail silently at runtime — the corruption is not caught until the flow is tested live
- **Read-only incidents:** corruption has now been observed during read-only sessions (Peek Code capture only, no canvas edits) — not only during structural edits as initially believed
- **Cross-flow pattern:** the same corruption pattern has been independently observed in a second, unrelated flow in the same environment, suggesting an environment-level or platform-level root cause rather than something specific to one flow's structure

---

## Incident log

| Date | Flow | Actions blanked | Trigger context |
|---|---|---|---|
| Early Aug 2026 | Flow B | ~6 | Canvas edits |
| ~15 Aug 2026 | Flow B | ~6 | Canvas edits |
| ~15 Aug 2026 | Email Triage | ~10 | Canvas edits |
| 21 Aug 2026 (morning) | Flow B | 22 | Canvas edits — **largest single incident at that point; dual-branch, both recurring and one-off branches affected simultaneously** |
| 21 Aug 2026 (afternoon) | Flow B | 1 (same action, `OF05c`, twice in one afternoon) | Canvas edits |
| 22 Aug 2026 (morning) | Flow B | 26 | Canvas edits |
| 22 Aug 2026 (evening) | Flow B | 26 | **Read-only — Peek Code capture only, no canvas edits made** |

**Total confirmed incidents:** 10+ across two flows, over approximately 3 weeks of active development.

**Total actions restored across all incidents:** estimated 100+.

---

## Reproduction pattern

Not reliably reproducible on demand, but correlates with:

1. Opening the flow in the Copilot Studio embedded Designer
2. Making structural edits (adding, moving, or deleting actions)
3. Save and reserialization events (Save draft, Publish)
4. **Occasionally: no canvas interaction at all** — most recent incident struck during a read-only Peek Code observation session with no edits made

The correlation with structural edits is strong but not exclusive. The read-only incident on 22 Aug suggests the corruption may be triggered by a background reserialization event rather than exclusively by user-initiated edits.

---

## Technical detail: what the corruption looks like in raw JSON

**Before corruption (correct):**
```json
{
  "type": "SetVariable",
  "inputs": {
    "name": "varFinalMatchCount",
    "value": "@string(outputs('Compose_Match_Count'))"
  },
  "runAfter": { ... }
}
```

**After corruption (blanked value field):**
```json
{
  "type": "SetVariable",
  "inputs": {
    "name": "varFinalMatchCount"
  },
  "runAfter": { ... }
}
```

The `value` key is absent entirely — not set to null or an empty string, but removed from the JSON object. The action's `name`, `type`, `runAfter`, and `metadata` are all intact.

**Observed variant:** on at least one occasion (`OF05c`, 21 Aug afternoon), the value was not fully removed but instead **partially truncated** — `@string(...)` was corrupted to `tring(...)`, causing a runtime error (`The template function 'tring' is not defined or not valid`) rather than a Flow Checker error. This suggests the corruption can affect the value at the character level, not just removing it wholesale.

---

## What we would like from Microsoft

1. **Is this a known platform bug?** If so, is there a fix in progress or a workaround available?

2. **Can flows be exported and edited externally** (e.g. via VS Code / Power Platform CLI / solution export) as a mitigation? This would allow us to edit flow JSON directly, bypassing the Designer as the corruption surface. We understand this approach exists for Power Automate standalone flows — it's less clear whether it's supported for flows embedded in Copilot Studio agent solutions.

3. **Is there an environment setting, connection configuration, or flow structure pattern** that is known to correlate with or prevent this corruption?

4. **Can Microsoft investigate this specific environment and flow** to determine whether there is a platform-level telemetry event that correlates with the corruption incidents listed above?

---

## Supporting material available

- Full Peek Code captures (before and after each incident) committed to GitHub repo `Crox2310/Teams-OneNote-Meeting-Capture`, folder `12-phase-2-validation/`
- Session notes for each incident with exact timestamps, action counts, and recovery steps
- `known-good-values-master-reference.md` — the manual reference document maintained specifically to enable recovery from this bug
- Second flow repo (`Crox2310/Copilot-Email-Triage-Agent`) with independent incident records

---

*Prepared 22 August 2026. Contact: David Croxson, Senior Head of Product, Sainsbury's Supply Chain Technology.*
