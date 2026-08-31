# CURRENT STATE — Teams-OneNote Meeting Capture (start here)

**Last updated:** 31 August 2026
**⚠️ New Claude instance: read these session notes first (most recent first):**

**31 August — Stage 1, Stage 2, and Stage 4 all closed in one session.**
- `session-2026-08-31-stage-2-and-stage-4.md` — **start here.** Stage 2 (date in opening utterance + date header in candidate list) built and gated. Stage 4 struck — already complete. Flow A and Topic published.
- `session-2026-08-31-stage-1-safety-net.md` — Stage 1 safety net (S1W01–S1W05 write-back chain) built, gated, UJ1–UJ5 regression passed. Flow B published.

**30 August — Stage 0. Four factual checks, no changes.**
- `findings-2026-08-30-stage-0-facts.md` — All four checks resolved or narrowed.

**29 August — design and review session. No flows changed.**
- `design-2026-08-29-target-state-and-backlog.md` — Target-state user journey, additive-contract rule, retention decision, and the ordered Stage 0–7 backlog.
- `analysis-2026-08-29-architecture-outside-view.md` — outside-view architecture review.
- `addendum-2026-08-29-contract-naming.md` — contract-layer naming rules.
- `build-narrative-log.md` — running record of decisions and lessons.

**28 August — chat capture scoping.**
- `design-flow-c-chat-transcript-capture.md` — Flow C design, agreed but not built.
- `handover-2026-08-28-recurring-chat-scoping.md` / `-graph-confirmed.md` / `-teams-chat-power-automate-confirmed.md` — chain proven; transcripts hard-blocked; DLP blocks Entra ID connector.

**23 August and earlier.**
- `session-2026-08-23-part3-fr01.md` — FR-01 resolved.
- `session-2026-08-23-part2-fr03-fr02-bug02.md` — FR-03, FR-02, BUG-02 all resolved.
- `session-2026-08-23-bug01-investigation-and-resolution.md` — BUG-01 resolved.

---

## TL;DR

**31 August — three stages closed.** Stage 1 (safety net), Stage 2 (date in opening prompt, date header in candidate list), Stage 4 (OutStatus surfacing — already done, struck). Flow A, Flow B, and Topic all published and validated.

**Next action: Stage 3 — Remove the redundant Flow A call.** See `design-2026-08-29-target-state-and-backlog.md`.

---

## What's confirmed working

| Item | Status |
|---|---|
| **Stage 1 — Safety net (S1W01–S1W05 write-back)** | ✅ Built, gated, and regression-tested 31 Aug |
| **Stage 2 — Date in opening utterance + candidate list header** | ✅ Built and gated 31 Aug |
| **Stage 4 — Six OutStatus values surfaced in Topic** | ✅ Already complete (23 Aug), confirmed and struck 31 Aug |
| **BUG-01 — Second-occurrence recurring capture** | ✅ Resolved and validated 23 Aug |
| **BUG-02 — Zero-match day navigation** | ✅ Resolved and validated 23 Aug |
| **FR-01 — Chronological candidate list ordering** | ✅ Live |
| **FR-02 — Holiday/leave/period/admin-block filter (11 patterns)** | ✅ Live |
| **FR-03 — OneNote link shortening (via hyperlink)** | ✅ Live |
| **Issue #1 — Per-occurrence recurring pages (FB-01–FB-05)** | ✅ Fully confirmed |
| **Issue #2 — Recapture content protection** | ✅ Confirmed live |
| **Issue #3 — Date entry format handling** | ✅ Confirmed live |
| **OutStatus differentiation (all 6 values + STALE_MAPPING)** | ✅ Live in Flow B and Topic |
| **FA16 defensive guard** | ✅ Confirmed live |
| **FA43 coalescing gap** | ✅ Fixed and published |
| **BadGateway fix (native Create item, status 201)** | ✅ Verified end-to-end |
| **SharePoint `SeriesMasterId` indexing** | ✅ Confirmed in place, accepts `$filter` cleanly |

## Remaining backlog (from `design-2026-08-29-target-state-and-backlog.md`)

| Stage | Item | Status |
|---|---|---|
| Stage 3 | Remove redundant Flow A call | Not started — next |
| Stage 5 | Perceived latency (S5.2 path, per S0.1 finding) | Not started |
| Stage 6 | Naming convention audit | Not started |
| Stage 7 | Child-flow extraction and Flow C | Not started, gated on S0.3 |

## Process debt

| Item | Priority | Notes |
|---|---|---|
| **Microsoft support ticket** | **Overdue — please submit** | `microsoft-discussion-brief-corruption-bug.md` — 12+ documented incidents across 3 flows. |
| Amendment log | Needs updating | 31 Aug Stage 2 changes not yet added. |
| Peek Code capture | Stale | Most recent full capture predates 23 Aug sprint. Needed as baseline for Stage 6. |

## Open questions

- **Do attendees appear in page content today?** Determines the size of the findability work.
- **Recurring-chat pagination gap** — still open, carried from 28 Aug. Close before Flow C is production-ready.
- **Does Flow B's `Create_OneNote_Page` use the same connector action tested in S0.1?** Check before Stage 5 build work starts.
- **Fix 1 partial** — `Filter_Pages_By_Title` inside `Apply_to_each_Existing_Section` still has an unguarded `formatDateTime` call on `text_5`. Lower risk path; deferred.

## Working-method notes

- **`if()` short-circuits in WDL** — confirmed 31 Aug via scratch test.
- **`MatchOptions.Contains & MatchOptions.IgnoreCase`** — confirmed valid Power Fx syntax in Copilot Studio (31 Aug).
- **Date parsing from `System.Activity.Text`** — preferred over `DateTimePrebuiltEntity` here because the entity approach prompts users who say a bare trigger phrase, breaking the existing-behaviour-unchanged requirement.
- **`PA - Scratch Diagnostics`** — always test unproven WDL expressions here before touching production.
- **Always re-pull fresh Peek Code** to verify a build step actually took.
- **All contract changes are additive** — add the new path, prove it, remove the old one as a separate change.
- **No dashes in action names** — em-dashes are the leading suspect for bulk corruption events.

## Where to look for detail

- **`session-2026-08-31-stage-2-and-stage-4.md`** — Stage 2 build, gate, and Stage 4 confirmation.
- **`session-2026-08-31-stage-1-safety-net.md`** — Stage 1 build, gate, and regression record.
- **`findings-2026-08-30-stage-0-facts.md`** — Stage 0 results.
- **`design-2026-08-29-target-state-and-backlog.md`** — target state and ordered backlog. The operative planning document.
- **`known-good-values-master-reference.md`** — Flow B reference (current as of Stage 1).
- **`known-good-values-flow-a-reference.md`** — Flow A reference (⚠️ needs Stage 2 FA40 change added).
- **`microsoft-discussion-brief-corruption-bug.md`** — ready to submit.

---
*Update at the end of each significant session.*
