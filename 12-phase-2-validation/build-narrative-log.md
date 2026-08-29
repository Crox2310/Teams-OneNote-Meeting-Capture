# Build narrative log

**Started:** 29 August 2026
**Purpose:** a running record of *decisions and what was learned from them*, kept deliberately separate from `amendment-log.md`. The amendment log records what changed in the flows. This records why, what constraint forced it, and what generalises — the material needed to explain later how this agent was built and what the method was.

**Primary intended use:** source material for a presentation on AI-assisted building methods.

---

## How to use this

Add an entry when a *decision* is made or a *constraint* is discovered, not when a bug is fixed. A bug fix belongs in `amendment-log.md`. A bug fix that changed how the project works belongs here too.

Entry shape:

```
### YYYY-MM-DD — short title
**Decision or discovery:** what was settled
**Why:** what forced it
**What generalises:** the transferable lesson, if any
**Evidence:** which document or run trace
```

Keep entries short. Detail lives in the dated documents; this is the spine.

---

## Recurring themes so far

These are the threads that run through the whole build and are the likely spine of any presentation. Each is evidenced by multiple entries below.

1. **Evidence before diagnosis.** Peek Code and Activity trace before proposing a fix; never reason from the Designer canvas or from assumption. Adopted after early sessions lost time to fixes built on guesses.
2. **A scratch flow as a safe test harness.** `PA - Scratch Diagnostics` exists so unproven expression syntax is never tested against production. On 23 August it caught a `sort()` syntax error that would otherwise have been the third guessed-wrong-syntax incident of the day.
3. **Known-good-values files as a recovery mechanism.** A platform corruption bug that blanks twenty-plus value fields at once is survivable only because the correct values are written down outside the platform.
4. **Design documents written before building.** Decisions settled in a dated document so a build session starts from a settled design rather than figuring it out mid-session.
5. **Short sessions with explicit handovers.** The project is carried across many short sessions by handover documents rather than by anyone holding it in their head.
6. **The platform, not the problem, shaped the architecture.** DLP policy, tenant settings, connector behaviour and an editor corruption bug drove more design decisions than the requirements did.

---

## Entries

### 2026-06 to 2026-07 — validation arc
**Decision or discovery:** all five user journeys (UJ1–UJ5) passed as a complete validated set for the first time on 20 July 2026, after an extended period of partial passes.
**Why:** P/N navigation defects spanned the Topic and Flow A simultaneously and could not be fixed from either side alone.
**What generalises:** in a three-component system, a defect that appears in one component is often a contract problem between two.
**Evidence:** `uj1`–`uj5-validation-record.md`, `2026-07-20-date-jump-feature-and-uj-validation.md`

### 2026-07/08 — the corruption pattern
**Decision or discovery:** value fields blank across twenty-plus `SetVariable` and `InitializeVariable` actions simultaneously, reproducibly, in the Power Automate Designer. Twelve-plus incidents across three flows.
**Why:** platform-level, not caused by any specific edit pattern that could be avoided.
**What generalises:** when the tool itself is the unreliable component, the response is external redundancy — reference files, small blast radius, verification after every change — not more careful editing.
**Evidence:** `microsoft-discussion-brief-corruption-bug.md`, `handover-2026-08-15-session2-corruption-mechanism-identified.md`

### 2026-08-20 — per-occurrence pages for recurring meetings
**Decision or discovery:** each occurrence of a recurring series gets its own dated page; a recapture on the same date appends to that date's page.
**Why:** field observation during live use — one page per series meant a second capture silently appended to the previous occurrence's page.
**What generalises:** the requirement was only visible once the system was in real daily use. No amount of design review would have surfaced it.
**Evidence:** `design-amendment-2026-08-20-per-occurrence-recurring-pages.md`

### 2026-08-23 — backlog cleared in a single day
**Decision or discovery:** BUG-01, BUG-02, FR-01, FR-02 and FR-03 all resolved and confirmed end to end in one day.
**Why:** the working method had matured — evidence first, scratch-test unproven syntax, re-pull Peek Code to verify every edit actually took.
**What generalises:** three real bugs were caught that day purely by re-pulling Peek Code after an edit rather than assuming a paste succeeded. Verification after every step is what allowed working at pace.
**Evidence:** `session-2026-08-23-*.md`, `CURRENT-STATE.md`

### 2026-08-23 — BUG-01 root cause was a schema constraint
**Decision or discovery:** the recurring-page overwrite traced to a unique constraint on `SeriesMasterId` in the SharePoint list.
**Why:** the schema still asserted "one page per series" after the design had moved to one page per occurrence.
**What generalises:** a uniqueness constraint on one column of what has become a composite key is a latent bug. When a key changes shape, the constraints must change with it.
**Evidence:** `session-2026-08-23-bug01-investigation-and-resolution.md`

### 2026-08-28 — two hard platform constraints confirmed
**Decision or discovery:** Graph API access to Teams meeting transcripts is disabled at tenant level, and the "HTTP with Microsoft Entra ID" connector is blocked tenant-wide by DLP policy. Native Teams connector actions work as an alternative.
**Why:** organisational policy, not permissions gaps or build errors.
**What generalises:** in an enterprise tenant, capability is bounded by policy as much as by API surface. Confirming a blocker as a hard stop is as valuable as solving it, because it prevents repeated attempts.
**Evidence:** `handover-2026-08-28-teams-chat-extraction-graph-confirmed.md`

### 2026-08-28 — Teams invite HTML is not a stable format
**Decision or discovery:** extraction keyed to a specific anchor id failed against a second meeting using a newer invite template. Replaced with a search for the join URL pattern itself.
**Why:** two invite templates in use simultaneously in the same tenant.
**What generalises:** parse for the thing you actually need, not for the scaffolding around it. The scaffolding is the vendor's to change.
**Evidence:** `handover-2026-08-28-recurring-chat-scoping.md`

### 2026-08-29 — outside-view architecture review
**Decision or discovery:** the architecture is emergent rather than designed. Reviewed against the question "how would this be built today, knowing everything now learned." Conclusions: flow boundaries should follow capability rather than user journey; the system should stay flow-heavy with model reasoning reserved for judgement tasks; the mapping list should become a cache rather than the source of truth; OneNote lane routing should not be automated.
**Why:** three months of incremental building had produced a working system whose shape was never chosen as a whole.
**What generalises:** an emergent architecture is not automatically wrong, but it is unexamined. The review is worth running once the system is stable enough that the answer can be acted on incrementally.
**Evidence:** `analysis-2026-08-29-architecture-outside-view.md`

### 2026-08-29 — strangler migration over clean-sheet rebuild
**Decision or discovery:** the target architecture will be reached incrementally, with the agent usable throughout. All contract changes additive: add the new path, prove it, remove the old one separately.
**Why:** the value in the current system is the validation, not the flow definitions. A rebuild discards three months of hard-won knowledge and does not escape the corruption pattern, which affects any flow of similar size.
**What generalises:** when the accumulated asset is *knowledge about the system's failure modes* rather than the code itself, rewriting destroys more than it creates.
**Evidence:** `design-2026-08-29-target-state-and-backlog.md`

### 2026-08-29 — target state defined as a user journey, not an architecture
**Decision or discovery:** the target state is expressed as a four-step user journey (down from six), with the backlog ordered to deliver that journey rather than to deliver the architecture.
**Why:** the two costly steps in the current journey — a redundant calendar fetch on selection, and an unexplained five-second wait — are invisible to the user as costs and are experienced only as slowness.
**What generalises:** mapping the journey against the machinery underneath made visible that the capability built is wider than the capability experienced. Flow B differentiates six status values; the Topic collapses them into two messages.
**Evidence:** `design-2026-08-29-target-state-and-backlog.md`

---
*Add entries at the point of decision, not retrospectively. Retrospective entries lose the reasoning, which is the part worth keeping.*
