# CURRENT STATE — Teams-OneNote Meeting Capture (start here)

**Last updated:** 22 August 2026, end of day (full day session)
**⚠️ New Claude instance: read these session notes first:**
- `session-2026-08-22-backlog-reduction-and-fb04-confirmed.md` — morning
- `session-2026-08-22-afternoon-addendum.md` — multi-occurrence verification + BadGateway pattern
- `session-2026-08-22-outstatus-differentiation.md` — OutStatus full implementation

---

## TL;DR

**Exceptionally productive day.** All three field-reported issues confirmed live. Five backlog items closed in the morning (character-gap fix, FA16 guard, link-format fix, FB-05, FA33A recovery). Issue #1 fully verified across multiple series and multiple occurrence dates. `OutStatus` differentiation (the highest-priority remaining gap from 20 July) implemented across all six values and confirmed live. Remaining work is UJ3–5 gaps, process debt, and one intermittent infrastructure issue — none blocking normal usage.

---

## What's confirmed working

| Item | Status |
|---|---|
| **Issue #1 — Per-occurrence recurring pages (FB-01–FB-05)** | **✅ Fully confirmed, multiple series, multiple dates** |
| **Issue #2 — Recapture content protection** | **✅ Confirmed live** |
| **Issue #3 — Date entry format handling** | **✅ Confirmed live** |
| **OutStatus differentiation (all 6 values)** | **✅ Confirmed live, 22 Aug** |
| Character-gap fix (section name sanitiser) | ✅ Published 22 Aug |
| FA16 defensive guard | ✅ Published 22 Aug (not yet verified by live run) |
| Link-format bug fix | ✅ Published 22 Aug |
| FB-05 dated page title fix | ✅ Confirmed live 22 Aug |
| FA33A corruption recovery | ✅ Published 22 Aug |
| Get_items — no structural fault | ✅ Confirmed — transient only |

## Known intermittent infrastructure issue

**`Send an HTTP request to SharePoint` (mapping row write) intermittently returns 502 BadGateway.** Not a flow logic fault. Now surfaces as `PARTIAL_SUCCESS` in `OutStatus` rather than silently reporting success. Workaround: retry the capture. To be added to Microsoft support ticket.

---

## Remaining backlog

### Build work
| Item | Priority | Notes |
|---|---|---|
| UJ3 — stale-row / duplicate-row detection | Medium | Not yet built. |
| UJ4 — section choice, blank-SeriesMasterId fallback, SectionRetryCount | Medium | Not yet built (3 gaps). |
| UJ5 — reword/retry, explicit Stop | Low | Not yet built (2 gaps). |
| FA43 IsRecurring/SeriesMaster coalescing gap | Low | Open since July. |
| BadGateway — switch to native Create item connector | Medium | Replace `Send an HTTP request to SharePoint` with native SharePoint `Create item` action for better throttling resilience. |
| Flow A solution-aware / VS Code editable | Low | One-time step, not yet started. |

### Unverified by live run
| Item | Priority | Notes |
|---|---|---|
| FA16 defensive guard | Low | Published, needs a live run to confirm. |

### Process debt
| Item | Priority | Notes |
|---|---|---|
| **Microsoft support ticket** | **Overdue** | Still not submitted. 10+ corruption incidents + BadGateway pattern. |
| Amendment log backfill | Low | Never been used. |
| `known-good-values-master-reference.md` update | Low | Needs FB-01/02/04/05 + OutStatus expressions added. |
| Orphaned test pages in OneNote | Housekeeping | A few stale pages from testing. |

## Recommended next session

1. **Microsoft support ticket** — Sonnet 4.6, Standard. Overdue.
2. **BadGateway fix** — replace `Send an HTTP request to SharePoint` with native `Create item` connector. Sonnet 4.6, Standard.
3. **UJ3 stale-row detection** — Sonnet 4.6, Standard.

## Where to look for detail

- **`session-2026-08-22-outstatus-differentiation.md`** — OutStatus full design and implementation.
- **`session-2026-08-22-afternoon-addendum.md`** — multi-occurrence verification, BadGateway pattern.
- **`session-2026-08-22-backlog-reduction-and-fb04-confirmed.md`** — morning session, FB-04/05 confirmation.
- **`flow-reference-2026-08-21-full-peek-code-capture.md`** — last full Peek Code snapshot (pre-today's structural additions — a fresh capture is due).

---
*Update at the end of each significant session.*
