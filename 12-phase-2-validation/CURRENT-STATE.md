# CURRENT STATE — Teams-OneNote Meeting Capture (start here)

**Last updated:** 29 August 2026
**⚠️ New Claude instance: read these session notes first (most recent first):**

**29 August — design and review session. No flows changed.**
- `design-2026-08-29-target-state-and-backlog.md` — **start here for what to do next.** Target-state user journey, the additive-contract rule, retention decision, and the ordered Stage 0–7 backlog with a test gate on each stage.
- `analysis-2026-08-29-architecture-outside-view.md` — outside-view architecture review. Flow boundaries, agents vs flows, the case against automating OneNote lane routing, state and coordination, critique of the Flow C design, and structural performance findings.
- `addendum-2026-08-29-contract-naming.md` — extends the 23 Aug naming audit to the `text_n` contract layer, and argues for doing it first via additive migration rather than last.
- `build-narrative-log.md` — running record of decisions and lessons, kept separate from `amendment-log.md`. Source material for the build-method presentation.

**28 August — chat capture scoping.**
- `design-flow-c-chat-transcript-capture.md` — Flow C design, agreed but not built.
- `handover-2026-08-28-recurring-chat-scoping.md` — invite-template variance, recurring chat scoping, and the open pagination gap.
- `handover-2026-08-28-teams-chat-power-automate-confirmed.md` / `-graph-confirmed.md` — chain proven; transcripts hard-blocked at tenant level; DLP blocks the Entra ID HTTP connector.
- `design-idea-2026-08-28-onenote-lane-routing-via-category.md` — idea captured. See the 29 Aug review before building this.

**23 August and earlier.**
- `session-2026-08-23-part3-fr01.md` — **FR-01 resolved.** Chronological candidate list ordering, new WDL `sort()` syntax discovered safely via scratch flow.
- `session-2026-08-23-part2-fr03-fr02-bug02.md` — **FR-03, FR-02, and BUG-02 all resolved.** Link now a clean hyperlink; 11-pattern candidate list filter live; zero-match-day navigation gap found and fixed.
- `session-2026-08-23-bug01-investigation-and-resolution.md` — **BUG-01 resolved.** Flow A corruption (first ever), Flow B 21-action corruption, `Set_varOutStatus` paren typo, and the true root cause: `SeriesMasterId` unique-constraint on the SharePoint list.
- `session-2026-08-22-evening-uj345.md` — evening session: UJ3/4/5 work
- `session-2026-08-22-badgateway-verification.md` — BadGateway fix verified
- `session-2026-08-22-fa43-and-endofday.md` — FA43 coalescing fix
- `session-2026-08-22-fa16-and-badgateway-fix.md` — FA16 verified, BadGateway native connector fix
- `session-2026-08-22-outstatus-differentiation.md` — OutStatus all 6 values
- `session-2026-08-22-afternoon-addendum.md` — multi-occurrence verification
- `session-2026-08-22-backlog-reduction-and-fb04-confirmed.md` — morning session

---

## TL;DR

**23 August — a complete, exceptional day. Every item raised (all backlog carry-overs and everything discovered live) is now resolved.** BUG-01 (second-occurrence recurring capture overwrite) fully root-caused and fixed. First-ever Flow A corruption incident handled. FR-03 (link shortening) resolved via markdown hyperlink. FR-02 (holiday/leave/period/admin-block filter) built and live, 11 patterns, three real bugs caught and fixed during the build. BUG-02 (zero-match day navigation gap) discovered and fixed same-day. FR-01 (chronological ordering) confirmed as a genuine bug via live evidence, built, and fixed — with the correct WDL `sort()` syntax discovered safely in the scratch flow rather than against production.

**28–29 August — scope moved to chat capture and then to architecture.** Flow C is designed but not built. A full outside-view architecture review was run on 29 August and produced a defined target state plus an ordered backlog. **Nothing was changed in Flow A, Flow B, the Topic, or the SharePoint list during that session.**

**Next action: Stage 0 of the backlog — four factual checks, no changes.** See `design-2026-08-29-target-state-and-backlog.md`.

---

## What's confirmed working

| Item | Status |
|---|---|
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
| **UJ5b/UJ5a/UJ4b/UJ3** | ✅ Published (22 Aug evening) |
| Character-gap fix, link-format fix, FB-05 | ✅ Published |

## Remaining backlog — explicitly not urgent

| Item | Notes |
|---|---|
| UJ3b — Automatic stale-row cleanup | Resilience/edge-case hardening, not fixing anything currently broken. Overlaps with the 30-day retention decision in the 29 Aug backlog. |
| UJ4a — Section choice disambiguation | No design work started. Same category as above. |
| UJ4c — SectionRetryCount retry loop | Higher corruption risk (Do Until shape) than the other two. |
| `Condition_Should_Write_Mapping` explicit guard | Defense-in-depth candidate, root cause already fixed upstream. |
| Flow A solution-aware / VS Code editable | One-time setup step, not started. **Now blocking:** Stage 7 of the 29 Aug backlog depends on whether child flows are available. |

### Process debt
| Item | Priority | Notes |
|---|---|---|
| **Microsoft support ticket** | **Overdue — please submit** | `microsoft-discussion-brief-corruption-bug.md` — now has 12+ documented incidents across 3 flows. Has been ready for weeks and repeatedly deferred. |
| Amendment log | Needs updating | Full 23 August change set (BUG-01, Flow A corruption, FR-03, FR-02, BUG-02, FR-01) not yet added. |
| Peek Code capture is stale | Worth refreshing | The most recent full capture (`flow-reference-2026-08-21-...`) predates the 23 Aug sprint. The 29 Aug review leans on it and flags this explicitly. A fresh capture is also the required baseline for any rename pass. |

## Open questions raised 29 August

- **Does the Topic surface the six OutStatus values?** In the 21 Aug Topic YAML, `C11_Check_OutStatus` branches on `OutStatus = "OK"` against one generic error message. If the Topic was not updated alongside Flow B on 23 Aug, the six values collapse into two on the way out. Check before starting Stage 4.
- **Do attendees appear in page content today?** Determines the size of the findability work.
- **Recurring-chat pagination gap** — still open, carried from 28 Aug. Should close before Flow C is treated as production-ready.

## Working-method notes worth carrying forward

- **`PA - Scratch Diagnostics`** continues to be the right default for testing any WDL expression syntax that isn't already proven in this codebase — caught the `sort()` lambda-syntax error safely on 23 Aug, avoiding a live production error.
- **WDL does not support arrow-function/lambda syntax** anywhere encountered so far (`select()`, `isMatch` doesn't exist at all — that's Power Fx only, `sort()` takes a plain string key name, not an expression). Worth checking any new WDL function's exact signature against real evidence before assuming JS/Power-Fx-style syntax will work.
- **Always re-pull fresh Peek Code to verify a build step actually took**, rather than assuming a paste or edit succeeded — caught three real bugs on 23 Aug this way.
- **All contract changes are additive from here on** (29 Aug decision): add the new path, prove it, remove the old one as a separate change. The agent stays usable throughout.

## Recommended next session

1. **Stage 0 of the 29 August backlog** — four factual checks, no changes to anything. Roughly one short session. Two of the four determine the shape of later work.
2. **Submit the Microsoft support ticket** — still the only outstanding "should do soon" item from the 23 Aug list.
3. Update `amendment-log.md` with the full 23 August change set.

## Where to look for detail

- **`design-2026-08-29-target-state-and-backlog.md`** — target state, backlog, test gates. The operative planning document.
- **`analysis-2026-08-29-architecture-outside-view.md`** — the reasoning behind the backlog, including disagreements with decisions already documented.
- **`build-narrative-log.md`** — decisions and lessons, for the build-method presentation.
- **`known-good-values-master-reference.md`** — Flow B reference, includes resolved BUG-01 write-up and corrected `Set_varOutStatus` expression.
- **`known-good-values-flow-a-reference.md`** — Flow A reference (created 23 Aug — worth updating with FA09B/FA09C additions from FR-02/FR-01).
- **`microsoft-discussion-brief-corruption-bug.md`** — ready to submit, needs the 23 Aug incidents added to its table first.

---
*Update at the end of each significant session.*
