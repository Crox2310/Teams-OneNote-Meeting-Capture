# CURRENT STATE — Teams-OneNote Meeting Capture (start here)

**Last updated:** 23 August 2026
**⚠️ New Claude instance: read these session notes first (most recent first):**
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

**23 August session: BUG-01 fully resolved and validated end-to-end.** Root cause was three-fold: corrupted `varFinal*` variables (platform corruption pattern), a paren-balance typo in the known-good reference doc, and — the real structural cause — a SharePoint "Enforce unique values" constraint on `SeriesMasterId` that blocked any second occurrence of a recurring series from ever getting its own mapping row, independent of flow logic. Also: Flow A hit by the corruption pattern for the first time (previously only Flow B and Email Triage), recovered and a new `known-good-values-flow-a-reference.md` created.

---

## What's confirmed working

| Item | Status |
|---|---|
| **BUG-01 — Second-occurrence recurring capture** | **✅ Resolved and validated 23 Aug** |
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
| FR-02 — Holiday/leave filter | **Next up** | Low complexity, high practical value. Queued from 22 Aug. |
| FR-01 — Chronological candidate list ordering | Medium | Confirm current Graph API ordering behaviour first. |
| FR-03 — Link shortening | Low-Medium | Needs option evaluation (SharePoint short links vs OneNote deep link vs display text). |
| UJ3b — Automatic stale-row cleanup | Medium | Lower urgency now the unique-constraint fix removes the main failure mode, but still valuable — a live orphaned-row example was found and manually cleaned up on 23 Aug. |
| UJ4a — Section choice disambiguation | Medium | Not yet built. |
| UJ4c — SectionRetryCount retry loop | Low | Not yet built. |
| `Condition_Should_Write_Mapping` explicit guard | Low | Defense-in-depth candidate flagged 23 Aug, not urgent — root cause fixed upstream. |
| Flow A solution-aware / VS Code editable | Low | One-time step, not yet started. |

### Process debt
| Item | Priority | Notes |
|---|---|---|
| **Microsoft discussion brief** | **Ready, consider submitting soon** | `microsoft-discussion-brief-corruption-bug.md` — 23 Aug added a 12th+ corruption data point (first-ever Flow A hit, plus a 21-action Flow B incident). |
| `known-good-values-flow-a-reference.md` | New, created 23 Aug | Companion to the Flow B master reference — Flow A previously had no dedicated doc. |
| Amendment log | Needs 23 Aug items added | |

## Recommended next session

1. **FR-02 holiday/leave filter** — low complexity, ready to build.
2. **FR-01 candidate list ordering** — confirm current behaviour, likely low complexity fix if needed.
3. **UJ3b automatic stale-row cleanup** — now lower urgency but still good resilience work.
4. Consider finally submitting the Microsoft support ticket given today's additional corruption data.

## Where to look for detail

- **`session-2026-08-23-bug01-investigation-and-resolution.md`** — today's full investigation and fix chain.
- **`known-good-values-master-reference.md`** — Flow B reference, now includes a resolved BUG-01 write-up and the corrected `Set_varOutStatus` expression.
- **`known-good-values-flow-a-reference.md`** — new Flow A reference.
- **`session-2026-08-22-evening-uj345.md`** — previous evening's session.
- **`microsoft-discussion-brief-corruption-bug.md`** — ready for Microsoft meeting.

---
*Update at the end of each significant session.*
