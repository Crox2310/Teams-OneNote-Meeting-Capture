# CURRENT STATE — Teams-OneNote Meeting Capture (start here)

**Last updated:** 22 August 2026, end of evening session
**⚠️ New Claude instance: read these session notes first (most recent first):**
- `session-2026-08-22-evening-uj345.md` — evening session: UJ3/4/5 work
- `session-2026-08-22-badgateway-verification.md` — BadGateway fix verified
- `session-2026-08-22-fa43-and-endofday.md` — FA43 coalescing fix
- `session-2026-08-22-fa16-and-badgateway-fix.md` — FA16 verified, BadGateway native connector fix
- `session-2026-08-22-outstatus-differentiation.md` — OutStatus all 6 values
- `session-2026-08-22-afternoon-addendum.md` — multi-occurrence verification
- `session-2026-08-22-backlog-reduction-and-fb04-confirmed.md` — morning session

---

## TL;DR

**Exceptional full day + evening session.** All three field-reported issues confirmed live. Major backlog reduction across the full day. UJ5a/b, UJ4b, UJ3 detection all published this evening. Remaining: UJ3b (automatic stale cleanup), UJ4a (section disambiguation), UJ4c (retry loop), and a planned investigation into recurring meeting capture errors.

---

## What's confirmed working

| Item | Status |
|---|---|
| **Issue #1 — Per-occurrence recurring pages (FB-01–FB-05)** | **✅ Fully confirmed** |
| **Issue #2 — Recapture content protection** | **✅ Confirmed live** |
| **Issue #3 — Date entry format handling** | **✅ Confirmed live** |
| **OutStatus differentiation (all 6 values + STALE_MAPPING)** | **✅ Confirmed live** |
| **FA16 defensive guard** | **✅ Confirmed live** |
| **FA43 coalescing gap** | **✅ Fixed and published** |
| **BadGateway fix (native Create item, status 201)** | **✅ Verified end-to-end** |
| **UJ5b — Explicit Cancel at selection prompt** | **✅ Published** |
| **UJ5a — Retry option on error** | **✅ Published** |
| **UJ4b — Blank SeriesMasterId guard** | **✅ Published** |
| **UJ3 — Stale-row detection (STALE_MAPPING)** | **✅ Published** |
| Character-gap fix, link-format fix, FB-05 | ✅ Published |

## Remaining backlog

### Build work
| Item | Priority | Notes |
|---|---|---|
| UJ3b — Automatic stale-row cleanup | Medium | When STALE_MAPPING detected, delete/update the stale mapping row so next capture takes CREATE_REQUIRED path automatically. Requires structural Flow B addition. |
| UJ4a — Section choice disambiguation | Medium | If section filter returns >1 result, Apply_to_each updates multiple pages. Disambiguation logic not yet built. |
| UJ4c — SectionRetryCount retry loop | Low | No retry if section creation fails transiently. |
| Recurring meeting capture errors investigation | **Next session priority** | David observing occasional errors on 2nd/3rd captures from same series. Diagnose from Activity trace evidence before building anything. |
| Flow A solution-aware / VS Code editable | Low | One-time step, not yet started. |

### Process debt
| Item | Priority | Notes |
|---|---|---|
| **Microsoft discussion brief** | **Ready** | `microsoft-discussion-brief-corruption-bug.md` committed — ready for in-person Microsoft meeting next week. |
| Amendment log | Up to date as of afternoon session | Needs evening session items added. |
| `known-good-values-master-reference.md` | Needs STALE_MAPPING expression update | |

## Recommended next session

1. **Investigate recurring meeting capture errors** — pull Activity trace from a failing capture, diagnose root cause before building anything. Sonnet 4.6, Standard.
2. **UJ3b automatic stale-row cleanup** — once root cause of recurring errors is understood, this may be related.
3. **UJ4a section disambiguation** — after recurring errors resolved.

## Where to look for detail

- **`session-2026-08-22-evening-uj345.md`** — this evening's session.
- **`microsoft-discussion-brief-corruption-bug.md`** — ready for Microsoft meeting.
- **`session-2026-08-22-badgateway-verification.md`** and earlier — full day session detail.

---
*Update at the end of each significant session.*
