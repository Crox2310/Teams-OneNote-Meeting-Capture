# Target state and delivery backlog — 29 August 2026

**Date:** 29 August 2026
**Status:** Design agreed in session, nothing built. This document defines the target user experience and the ordered backlog to reach it.
**Companion to:** `analysis-2026-08-29-architecture-outside-view.md` (the reasoning behind these choices) and `addendum-2026-08-29-contract-naming.md` (the naming rules the backlog assumes).

---

## 1. The governing constraint

**The agent must remain usable day to day throughout.** There is no window where the system is down for a rebuild.

This has a sharper implication than it first appears: **every contract change must be additive.** Adding an optional field to a flow trigger is safe, and has already been done once — `text_5` was deliberately left out of `required`. Renaming or removing a field means the Topic and the flow must be published in the same instant, or the agent is broken in between.

**Standing rule for all work below: add the new path alongside the old, prove it, then delete the old one as a separate change. Never a swap.**

This also settles the clean-sheet question. A full rebuild would discard the thing of greatest value — the validation, not the flow definitions. Five user journeys passed as a set, twelve-plus corruption incidents diagnosed, BUG-01 root-caused to a SharePoint unique constraint, the BadGateway workaround, the OneNote indexing race, the invite-template variance. A rebuild re-earns all of it and does not escape the corruption pattern, which is a Designer-level platform bug that affects any flow of similar size. **The approach is strangler migration, not big bang:** build the new small flows first, point new work at them, cut the old flow over once proven.

---

## 2. Target-state user journey

Six steps become four. The three costly steps are removed rather than hidden.

| # | What the user does | What runs underneath |
|---|---|---|
| 1 | Asks, including a date — "capture notes for yesterday" | Topic resolves the date from an entity, defaults to today when absent |
| 2 | Reads the candidate list; header confirms the resolved date | Flow A runs once; candidates held in a Topic variable |
| 3 | Picks a number, or moves a day with P, N or a date | Resolved in the Topic — no second calendar call |
| 4 | Gets the link, or a specific next step | Flow B responds early; all six statuses surfaced distinctly |

**Later extension, not in this backlog:** if the opening utterance also names the meeting ("capture the portfolio review from yesterday") and resolves to exactly one confident match, the candidate list is skipped entirely and the journey collapses to two steps. The list becomes the fallback for ambiguity rather than the default path. Worth building only after step 1 is proven in real use.

---

## 3. Design decisions settled by this document

**Natural-language date in the opening prompt.** The trigger accepts a date in the utterance and defaults to today when none is given, so existing behaviour is unchanged for a bare "capture". The proven pattern for the hard part already exists in the Email Triage Agent: `DateTimePrebuiltEntity` piped into a Text-typed variable, specifically to avoid the DateTime/String mismatch. `C1_Set_DateContext` already stores `yyyy-MM-dd`, so the shape matches.

**The resolved date is echoed in the candidate list header** — "Meetings for Thu 28 Aug". Natural-language date parsing will occasionally land on the wrong day; if the header states what was heard, a misparse is visible instead of silent.

**Mapping list retention: 30 days, keyed on `OccurrenceDate`, not on row creation date, deleted by a weekly scheduled flow.** Roughly 50–60 rows steady state.

Seven days was considered and rejected. While the list is the source of truth, an expired row for a meeting that still exists in OneNote produces a silent duplicate page rather than an error — so a week of leave would mean everything captured beforehand expires while away, and the first recapture on return quietly duplicates. Thirty days survives a fortnight's leave.

**Retention is not the fix for the 100-row limit.** That is a query problem with a query fix: server-side `$filter` on `SeriesMasterId`, or pagination enabled on `Get_items`. Deleting data to keep a query working is the kind of fix that looks fine until the data is needed for something else.

**The mapping list stays a lean transient cache and is not enriched into a retrieval index.** The durable index — lane, attendees, summary — lives as a rolling page in OneNote instead, next to the notes. Enriching the list and expiring its rows are incompatible; this is the choice between them.

---

## 4. Backlog

Ordered so nothing depends on an unanswered question, and each stage is independently revertible. Every stage carries an explicit test gate, following the method that worked through the 22–23 August backlog.

### Stage 0 — Facts. No changes to anything.

| Item | Question | Method |
|---|---|---|
| S0.1 | Does `<title>` in posted HTML set the page title at creation? | Scratch flow, disposable test notebook |
| S0.2 | Does the Teams connector's Graph HTTP action reach OneNote endpoints? | Scratch flow |
| S0.3 | Is the environment solution-aware enough for child flows, and does Copilot Studio `InvokeFlowAction` work against them? | Environment check |
| S0.4 | Why did `Get_items` return `[]` on 21 August against a populated list? Is `SeriesMasterId` an indexed column that will accept an OData filter? | List Settings plus a live run |

**Gate:** all four answered in one dated findings document. S0.2 and S0.3 change the *shape* of later stages rather than the detail, so nothing structural begins before this. Estimated one short session.

### Stage 1 — Safety net. No user-visible change.

- **S1.1** Wire `Filter_Pages_By_Title` into the create path so a missing mapping row falls back to a OneNote title lookup instead of silently creating a duplicate page.
- **S1.2** Add server-side `$filter` or enable pagination on `Get_items`, per S0.4's answer.

**Gate:** delete a mapping row for an already-captured meeting, recapture, confirm it appends rather than duplicates. Then a full UJ1–UJ5 regression pass. This stage goes first because it is what makes retention — and every later change — safe to get wrong.

### Stage 2 — Date in the opening prompt.

- **S2.1** Datetime entity on the trigger, resolved to a Text variable via the Email Triage Agent pattern, defaulting to today.
- **S2.2** Resolved date echoed in the candidate list header.
- **S2.3** Existing P/N/date question left exactly as it is. Purely additive.

**Gate:** five utterances including one deliberately ambiguous, plus a bare "capture" to confirm existing behaviour is untouched.

### Stage 3 — Remove the redundant Flow A call.

- **S3.1** Flow A adds a JSON candidate array to its response, alongside every field it already returns.
- **S3.2** Topic holds it and resolves the selection by index.
- **S3.3** Only once proven, remove the second `InvokeFlowAction`.

**Gate:** selection and navigation tested across recurring, one-off, and a zero-match day.

### Stage 4 — Surface the six statuses.

- **S4.1** Replace the Topic's binary `OutStatus = "OK"` branch with distinct, actionable messages for all six values plus `STALE_MAPPING`.

**Gate:** force each status and confirm the message. Cheapest user-experience win on the list — Flow B already computes the values.

### Stage 5 — Perceived latency.

- **S5.1** If S0.1 confirmed `<title>` works: remove `Set_PageTitle_Recurring`, the five-second delay, and the post-create page lookup chain.
- **S5.2** If not: move the Response action up to immediately after page creation, leaving the mapping write to complete behind it. Note this changes what "success" means and must be reconciled with Stage 4's status semantics.

**Gate:** timed comparison on the same meeting before and after, using Activity trace per-action durations plus wall-clock total.

### Stage 6 — Naming.

Its own dedicated session, per `analysis-scope-2026-08-23-naming-convention-audit.md`, with the contract-layer changes from `addendum-2026-08-29-contract-naming.md` done first as an additive migration.

### Stage 7 — Structure.

Child-flow extraction (Resolve Page, Write Region) and Flow C, gated on S0.3. Flow C points at the new child flows from the start, since it is new and has nothing to regress.

---

## 5. Sequencing trade accepted

User-experience work is deliberately ahead of the architectural split, because that is where the stated interest lies. The cost: Stage 3 changes Flow A's output contract, and if the child-flow split later moves that logic, it gets touched twice. Accepted, because Stage 7 is gated on an unknown (S0.3) and the user-experience stages are not.

---

## 6. Open questions

- Whether the Topic's `C11` condition was updated alongside Flow B's OutStatus differentiation on 23 August. If it was, Stage 4 is already complete and should be struck.
- Whether attendees currently appear in page content. Determines the size of the findability work.
- Whether existing pages created before the four-section template need migration, or simply gain the sections on next update. Carried over unresolved from `design-flow-c-chat-transcript-capture.md`.
- The recurring-chat pagination gap, carried over from `handover-2026-08-28-recurring-chat-scoping.md`. Should close before Flow C is treated as production-ready.

---
*Written 29 August 2026. No flow, Topic, or list was modified in producing this document.*
