# Addendum — contract-layer naming

**Date:** 29 August 2026
**Status:** Proposal. Nothing renamed.
**Extends:** `analysis-scope-2026-08-23-naming-convention-audit.md`. That document is not superseded — its assessment of action naming, its rename ordering, and its insistence on a fresh Peek Code baseline before any rename all stand. This addendum covers a layer it does not address, and argues for one change to its sequencing.

---

## 1. The layer the audit does not cover

The audit assesses action `displayName`s across Flow A, Flow B, and the Topic. It does not cover the **contract** — the trigger field names and Response field names that cross flow boundaries.

That layer is currently in worse shape than any of the three assessed. Flow B's trigger schema is positional and untyped: `text`, `text_1`, `text_2`, `text_3`, `text_4`, `text_5`, where `text` is `IsRecurring` and `text_1` is `MeetingTitle` for no reason that could be reconstructed by anyone reading it cold. The `title` attributes in the schema carry the real meaning, but every expression in the flow references the positional name.

This is what invited the field-swap slip caught on 23 August, and it gets worse with every field added. Flow C will add more.

---

## 2. Proposed conventions

**Flow-internal actions** — `F{X}##_Verb_Noun`, numbered in execution order, branch context in the noun rather than a bare numeric suffix. This is the audit's own proposal and is adopted here unchanged. New flows adopt it from action one rather than being retrofitted.

**Contract fields** — `camelCase` semantic names inside a single JSON payload field: `occurrenceDate`, `seriesMasterId`, `isRecurring`, `meetingTitle`, `pageHtml`, `meetingId`. Never positional, never `text_n`.

**Topic nodes** — `C##_Verb_Noun` on `displayName` only. Leave the YAML `id` fields as auto-generated. Renaming them is lower value and higher risk, since `GotoAction.actionId` references them by exact string.

**Number-block reservation** — pick a flow's prefix before building it, and reserve number blocks by concern rather than numbering strictly sequentially. Flow A's numbering held up under fast iteration on 23 August precisely because FA09B and FA09C could be inserted without renumbering everything downstream. Leaving gaps deliberately is what makes that repeatable, and it is currently an accident rather than a rule.

---

## 3. Disagreement with the audit's sequencing

The audit sequences contract renames **last**, on the grounds that they are the highest blast radius — a Response field name change breaks the Topic's `output.binding` immediately, whereas an internal `displayName` change does not.

That reasoning is correct about the risk of a *rename*. It does not apply to an *additive migration*, and the contract layer is the one place an additive migration is straightforward:

1. Add a single `payload` field to Flow B's trigger, carrying JSON. Leave every existing `text_n` field in place and still populated.
2. Add a `Parse JSON` action at the top of Flow B. Have downstream expressions read from the parsed object where the field is present, falling back to the positional field where it is not.
3. Both paths coexist and both work. The agent is never broken.
4. Once the JSON path is proven across all five user journeys, stop populating the old fields in the Topic.
5. As a separate, later change, remove the unused `text_n` fields from the schema.

At no point is there a state where the Topic and the flow must be published simultaneously. Afterwards, adding a field costs nothing and cannot be positionally confused with another.

**Recommendation: do the contract migration first, before the `displayName` rename pass.** It carries less risk than the audit assumes, it removes a live defect source rather than only improving legibility, and it makes the subsequent rename pass smaller because several `_1`/`_2` variable names exist only to shuttle positional trigger fields around.

---

## 4. Unchanged from the audit

Everything else in `analysis-scope-2026-08-23-naming-convention-audit.md` holds:

- Fresh Peek Code capture of Flow A, Flow B, and Topic YAML as a pre-change baseline.
- Map every cross-reference before touching anything.
- One change or small batch, then Flow Checker, then Peek Code re-verification, then the next batch. Never a bulk find-and-replace.
- Update `known-good-values-master-reference.md`, `known-good-values-flow-a-reference.md`, and `amendment-log.md` in the same session, not as a follow-up.
- Full live end-to-end test before the pass is considered complete.
- Sequence deliberately against the Microsoft support ticket, since a renamed flow changes what any Microsoft-side investigation would see relative to the ticket's incident log.

---
*Written 29 August 2026. Nothing renamed. Treat as a proposal requiring agreement before any change.*
