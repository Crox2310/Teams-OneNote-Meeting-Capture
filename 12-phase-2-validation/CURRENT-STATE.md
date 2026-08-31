# CURRENT STATE — Teams-OneNote Meeting Capture (start here)

**Last updated:** 31 August 2026
**⚠️ New Claude instance: read these session notes first (most recent first):**

**31 August — Stage 1 built, gated, and UJ1–UJ5 regression passed.**
- `session-2026-08-31-stage-1-safety-net.md` — **start here.** Full build and gate record. Safety net (S1W01–S1W05 write-back chain) built and validated. All five user journeys green. Stage 1 closed.

**30 August — Stage 0 of the 29 August backlog. Four factual checks, no changes.**
- `findings-2026-08-30-stage-0-facts.md` — All four checks resolved or narrowed. S0.1 and S0.2 both fail (structural blockers). S0.3 split. S0.4 confirmed safe for Stage 1's `$filter` change.

**29 August — design and review session. No flows changed.**
- `design-2026-08-29-target-state-and-backlog.md` — Target-state user journey, additive-contract rule, retention decision, and the ordered Stage 0–7 backlog.
- `analysis-2026-08-29-architecture-outside-view.md` — outside-view architecture review.
- `addendum-2026-08-29-contract-naming.md` — contract-layer naming rules.
- `build-narrative-log.md` — running record of decisions and lessons.

**28 August — chat capture scoping.**
- `design-flow-c-chat-transcript-capture.md` — Flow C design, agreed but not built.
- `handover-2026-08-28-recurring-chat-scoping.md` — invite-template variance, recurring chat scoping, open pagination gap.
- `handover-2026-08-28-teams-chat-power-automate-confirmed.md` / `-graph-confirmed.md` — chain proven; transcripts hard-blocked at tenant level; DLP blocks the Entra ID HTTP connector.
- `design-idea-2026-08-28-onenote-lane-routing-via-category.md` — idea captured. See 29 Aug review before building.

**23 August and earlier.**
- `session-2026-08-23-part3-fr01.md` — FR-01 resolved.
- `session-2026-08-23-part2-fr03-fr02-bug02.md` — FR-03, FR-02, BUG-02 all resolved.
- `session-2026-08-23-bug01-investigation-and-resolution.md` — BUG-01 resolved.

---

## TL;DR

**31 August — Stage 1 complete.** Safety net built (S1W01–S1W05 write-back chain, `varOneOffMappingId` variable, Fix 1 null guard on `S1_Filter_Pages_By_Title_PreCreate`). Gate passed: one-off meeting, mapping row deleted, recaptured — page appended not duplicated, row recreated with all URL fields. UJ1–UJ5 regression all green. Flow B published.

**30 August — Stage 0 complete.** Four factual checks resolved, no changes made to any flow, Topic, or list.

**23 August — all backlog items resolved.** BUG-01, BUG-02, FR-01, FR-02, FR-03 all fixed and validated.

**Next action: Stage 2 — Date in the opening prompt.** See `design-2026-08-29-target-state-and-backlog.md`.

---

## What's confirmed working

| Item | Status |
|---|---|
| **Stage 1 — Safety net (S1W01–S1W05 write-back)** | ✅ Built, gated, and regression-tested 31 Aug |
| **BUG-01 — Second-occurrence recurring capture** | ✅ Resolved and validated 23 Aug |
| **BUG-02 — Zero-match day navigation** | ✅ Resolved and validated 23 Aug |
| **FR-01 — Chronological candidate list ordering** | ✅ Live |
| **FR-02 — Holiday/leave/period/admin-block filter (11 patterns)** | ✅ Live |
| **FR-03 — OneNote link shortening (via hyperlink)** | ✅ Live |
| **Issue #1 — Per-occurrence recurring pages (FB-01–FB-05)** | ✅ Fully confirmed |
| **Issue #2 — Recapture content protection** | ✅ Confirmed live |
| **Issue #3 — Date entry format handling** | ✅ Confirmed live |
| **OutStatus differentiation (all 6 values + STALE_MAPPING)** | ✅ Confirmed live in Flow B — ⚠️ see open question below re: whether the Topic surfaces them |
| **FA16 defensive guard** | ✅ Confirmed live |
| **FA43 coalescing gap** | ✅ Fixed and published |
| **BadGateway fix (native Create item, status 201)** | ✅ Verified end-to-end |
| **SharePoint `SeriesMasterId` indexing** | ✅ Confirmed already in place, accepts `$filter` cleanly (30 Aug) |

## Remaining backlog (from `design-2026-08-29-target-state-and-backlog.md`)

| Stage | Item | Status |
|---|---|---|
| Stage 2 | Date in the opening prompt | Not started |
| Stage 3 | Remove redundant Flow A call | Not started |
| Stage 4 | Surface the six OutStatus values in the Topic | Not started |
| Stage 5 | Perceived latency (S5.2 path, per S0.1 finding) | Not started |
| Stage 6 | Naming convention audit | Not started |
| Stage 7 | Child-flow extraction and Flow C | Not started, gated on S0.3 |

## Process debt

| Item | Priority | Notes |
|---|---|---|
| **Microsoft support ticket** | **Overdue — please submit** | `microsoft-discussion-brief-corruption-bug.md` — 12+ documented incidents across 3 flows. Ready for weeks. |
| Amendment log | Needs updating | 23 Aug change set and 31 Aug Stage 1 changes not yet added. |
| `known-good-values-master-reference.md` | Needs updating | Predates S1W01–S1W05 and `varOneOffMappingId`. Refresh before next corruption incident. |
| Peek Code capture | Stale | Most recent full capture predates 23 Aug sprint. Needed as baseline for Stage 6 naming pass. |

## Open questions

- **Does the Topic surface the six OutStatus values?** `C11_Check_OutStatus` may still branch on `OutStatus = "OK"`. Check before starting Stage 4.
- **Do attendees appear in page content today?** Determines the size of the findability work.
- **Recurring-chat pagination gap** — still open, carried from 28 Aug. Close before Flow C is production-ready.
- **Does Flow B's `Create_OneNote_Page` use the same connector action tested in S0.1?** Check before Stage 5 build work starts.
- **Fix 1 partial** — `Filter_Pages_By_Title` inside `Apply_to_each_Existing_Section` still has an unguarded `formatDateTime` call on `text_5`. Lower risk path; noted but deferred.

## Working-method notes

- **`if()` short-circuits in WDL** — confirmed 31 Aug via scratch test. When the true branch condition matches, the false branch expression is never evaluated. Safe to use for null-guarding.
- **`PA - Scratch Diagnostics`** — always test unproven WDL expressions here before touching production.
- **Always re-pull fresh Peek Code** to verify a build step actually took.
- **All contract changes are additive** — add the new path, prove it, remove the old one as a separate change.
- **No dashes in action names** — em-dashes in action names are the leading suspect for bulk corruption events. All new actions use underscore-only naming.

## Where to look for detail

- **`session-2026-08-31-stage-1-safety-net.md`** — Stage 1 build, gate, and regression record.
- **`findings-2026-08-30-stage-0-facts.md`** — Stage 0 results.
- **`design-2026-08-29-target-state-and-backlog.md`** — target state and ordered backlog. The operative planning document.
- **`analysis-2026-08-29-architecture-outside-view.md`** — reasoning behind the backlog.
- **`known-good-values-master-reference.md`** — Flow B reference (⚠️ needs updating with Stage 1 additions).
- **`known-good-values-flow-a-reference.md`** — Flow A reference.
- **`microsoft-discussion-brief-corruption-bug.md`** — ready to submit.

---
*Update at the end of each significant session.*
