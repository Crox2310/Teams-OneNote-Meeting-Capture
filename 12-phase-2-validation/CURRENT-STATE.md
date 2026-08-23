# CURRENT STATE — Teams-OneNote Meeting Capture (start here)

**Last updated:** 23 August 2026
**⚠️ New Claude instance: read these session notes first (most recent first):**
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

**23 August — a complete, exceptional day. Every item raised (all backlog carry-overs and everything discovered live) is now resolved.** BUG-01 (second-occurrence recurring capture overwrite) fully root-caused and fixed. First-ever Flow A corruption incident handled. FR-03 (link shortening) resolved via markdown hyperlink. FR-02 (holiday/leave/period/admin-block filter) built and live, 11 patterns, three real bugs caught and fixed during the build. BUG-02 (zero-match day navigation gap) discovered and fixed same-day. FR-01 (chronological ordering) confirmed as a genuine bug via live evidence, built, and fixed — with the correct WDL `sort()` syntax discovered safely in the scratch flow rather than against production, avoiding what would have been the third guessed-wrong-syntax incident of the day.

**What's left is explicitly the lower-priority tail: UJ3b/UJ4a/UJ4c (all resilience/edge-case hardening, nothing currently broken), and the still-unsubmitted Microsoft support ticket.**

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
| **OutStatus differentiation (all 6 values + STALE_MAPPING)** | ✅ Confirmed live |
| **FA16 defensive guard** | ✅ Confirmed live |
| **FA43 coalescing gap** | ✅ Fixed and published |
| **BadGateway fix (native Create item, status 201)** | ✅ Verified end-to-end |
| **UJ5b/UJ5a/UJ4b/UJ3** | ✅ Published (22 Aug evening) |
| Character-gap fix, link-format fix, FB-05 | ✅ Published |

## Remaining backlog — explicitly not urgent

| Item | Notes |
|---|---|
| UJ3b — Automatic stale-row cleanup | Resilience/edge-case hardening, not fixing anything currently broken. |
| UJ4a — Section choice disambiguation | No design work started. Same category as above. |
| UJ4c — SectionRetryCount retry loop | Higher corruption risk (Do Until shape) than the other two. |
| `Condition_Should_Write_Mapping` explicit guard | Defense-in-depth candidate, root cause already fixed upstream. |
| Flow A solution-aware / VS Code editable | One-time setup step, not started. |

### Process debt
| Item | Priority | Notes |
|---|---|---|
| **Microsoft support ticket** | **Overdue — please submit** | `microsoft-discussion-brief-corruption-bug.md` — now has 12+ documented incidents across 3 flows. Has been ready for weeks and repeatedly deferred. |
| Amendment log | Needs updating | Full 23 August change set (BUG-01, Flow A corruption, FR-03, FR-02, BUG-02, FR-01) not yet added. |

## Working-method notes worth carrying forward

- **`PA - Scratch Diagnostics`** continues to be the right default for testing any WDL expression syntax that isn't already proven in this codebase — caught the `sort()` lambda-syntax error safely today, avoiding a live production error.
- **WDL does not support arrow-function/lambda syntax** anywhere encountered so far (`select()`, `isMatch` doesn't exist at all — that's Power Fx only, `sort()` takes a plain string key name, not an expression). Worth checking any new WDL function's exact signature against real evidence before assuming JS/Power-Fx-style syntax will work.
- **Always re-pull fresh Peek Code to verify a build step actually took**, rather than assuming a paste or edit succeeded — caught three real bugs today (regex over-escaping, an invalid function, a field-swap slip) this way.

## Recommended next session

1. **Submit the Microsoft support ticket** — genuinely the only outstanding "should do soon" item left. Strong evidence base.
2. Update `amendment-log.md` with the full 23 August change set.
3. UJ3b/UJ4a/UJ4c remain available whenever there's appetite, but none are time-critical — pick up only if there's spare capacity.

## Where to look for detail

- **`session-2026-08-23-part3-fr01.md`** — FR-01 (chronological ordering) and the `sort()` syntax discovery.
- **`session-2026-08-23-part2-fr03-fr02-bug02.md`** — FR-03, FR-02, BUG-02.
- **`session-2026-08-23-bug01-investigation-and-resolution.md`** — BUG-01 investigation and fix chain.
- **`known-good-values-master-reference.md`** — Flow B reference, includes resolved BUG-01 write-up and corrected `Set_varOutStatus` expression.
- **`known-good-values-flow-a-reference.md`** — Flow A reference (created 23 Aug — worth updating with FA09B/FA09C additions from FR-02/FR-01 in a future session).
- **`microsoft-discussion-brief-corruption-bug.md`** — ready to submit, needs the 23 Aug incidents added to its table first.

---
*Update at the end of each significant session.*
