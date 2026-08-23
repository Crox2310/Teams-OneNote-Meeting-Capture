# CURRENT STATE — Teams-OneNote Meeting Capture (start here)

**Last updated:** 23 August 2026
**⚠️ New Claude instance: read these session notes first (most recent first):**
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

**23 August, full day: exceptional session.** Morning: BUG-01 (second-occurrence recurring capture overwrite) fully resolved — root cause was three-fold (corrupted `varFinal*` variables, a paren-balance typo in the reference doc, and a SharePoint unique-constraint on `SeriesMasterId`). Flow A hit by the corruption pattern for the first time. Afternoon: FR-03 (link shortening) resolved via a markdown hyperlink in the Topic rather than a shorter URL (none of Microsoft's OneNote API URL variants were actually shorter — checked with live data before assuming). FR-02 (holiday/leave/period-week/admin-block filter) built and live, catching 11 patterns — three real bugs found and fixed during the build (regex over-escaping, an invalid WDL function, and a field-swap slip), each caught by insisting on fresh Peek Code confirmation rather than assuming a build step succeeded. This surfaced **BUG-02** — a pre-existing gap where a zero-match day had no working P/N/date navigation — found, scoped, and fixed same session.

---

## What's confirmed working

| Item | Status |
|---|---|
| **BUG-01 — Second-occurrence recurring capture** | ✅ Resolved and validated 23 Aug |
| **BUG-02 — Zero-match day navigation** | ✅ Resolved and validated 23 Aug |
| **FR-03 — OneNote link shortening (via hyperlink)** | ✅ Live |
| **FR-02 — Holiday/leave/period/admin-block filter (11 patterns)** | ✅ Live |
| **Issue #1 — Per-occurrence recurring pages (FB-01–FB-05)** | ✅ Fully confirmed |
| **Issue #2 — Recapture content protection** | ✅ Confirmed live |
| **Issue #3 — Date entry format handling** | ✅ Confirmed live |
| **OutStatus differentiation (all 6 values + STALE_MAPPING)** | ✅ Confirmed live |
| **FA16 defensive guard** | ✅ Confirmed live |
| **FA43 coalescing gap** | ✅ Fixed and published |
| **BadGateway fix (native Create item, status 201)** | ✅ Verified end-to-end |
| **UJ5b/UJ5a/UJ4b/UJ3** | ✅ Published (22 Aug evening) |
| Character-gap fix, link-format fix, FB-05 | ✅ Published |

## Remaining backlog

### Build work
| Item | Priority | Notes |
|---|---|---|
| FR-01 — Chronological candidate list ordering | Next up | Confirm current Graph API ordering behaviour first — may need no fix. |
| UJ3b — Automatic stale-row cleanup | Low-Medium | Discussed 23 Aug: not fixing anything currently broken, resilience/edge-case hardening only. Not time-critical. |
| UJ4a — Section choice disambiguation | Low-Medium | No design work done yet. Same as above — not urgent. |
| UJ4c — SectionRetryCount retry loop | Low | Higher corruption risk than the other two (Do Until loop shape). Not urgent. |
| `Condition_Should_Write_Mapping` explicit guard | Low | Defense-in-depth candidate flagged 23 Aug, not urgent — root cause fixed upstream. |
| Flow A solution-aware / VS Code editable | Low | One-time step, not yet started. |

### Process debt
| Item | Priority | Notes |
|---|---|---|
| **Microsoft support ticket** | **Overdue — please submit** | `microsoft-discussion-brief-corruption-bug.md` — now has 12+ documented incidents across 3 flows. Has been ready for weeks. |
| Amendment log | Needs 23 Aug items added | Not done this session — BUG-01, Flow A corruption, FR-03, FR-02, BUG-02 all need entries. |
| `known-good-values-flow-a-reference.md` | Created 23 Aug | Companion to the Flow B master reference. |

## Recommended next session

1. **FR-01 candidate list ordering** — confirm current behaviour first; likely low complexity if a fix is even needed.
2. **Submit the Microsoft support ticket** — genuinely overdue at this point, strong evidence base now.
3. Update `amendment-log.md` with the full 23 August change set.
4. UJ3b/UJ4a/UJ4c remain available but are explicitly not urgent — pick up only if there's spare capacity, not as a priority.

## Where to look for detail

- **`session-2026-08-23-part2-fr03-fr02-bug02.md`** — today's afternoon work (FR-03, FR-02, BUG-02), including the three bugs caught during the FR-02 build and how each was diagnosed.
- **`session-2026-08-23-bug01-investigation-and-resolution.md`** — today's morning investigation and BUG-01 fix chain.
- **`known-good-values-master-reference.md`** — Flow B reference, includes resolved BUG-01 write-up and corrected `Set_varOutStatus` expression.
- **`known-good-values-flow-a-reference.md`** — Flow A reference.
- **`microsoft-discussion-brief-corruption-bug.md`** — ready to submit, needs the 23 Aug incidents added to its table first.

---
*Update at the end of each significant session.*
