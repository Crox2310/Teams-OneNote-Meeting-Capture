# CURRENT STATE — Teams-OneNote Meeting Capture (start here)

**Last updated:** 22 August 2026, end of day
**⚠️ If you are a new Claude instance picking this up, read these two session notes first:**
- `session-2026-08-22-backlog-reduction-and-fb04-confirmed.md` — morning session
- `session-2026-08-22-afternoon-addendum.md` — afternoon continuation and multi-occurrence verification

---

## TL;DR

**All three field-reported issues are fully fixed and confirmed live.** Issue #1 (per-occurrence recurring pages) is verified end-to-end across multiple series and multiple occurrence dates — one section per series, one dated page per occurrence, create and recapture both working. Issues #2 and #3 confirmed from 20–21 Aug. A significant backlog reduction was completed on 22 Aug (character-gap fix, FA16 guard, link-format fix, FB-05). Remaining work is `OutStatus` differentiation, UJ3–5 gaps, and process debt — none blocking normal usage.

---

## What's confirmed working right now

| Item | Status |
|---|---|
| **Issue #1 — Per-occurrence recurring pages (FB-01–FB-05)** | **✅ Fully confirmed, 22 Aug — multiple series, multiple dates** |
| **Issue #2 — Recapture content protection** | **✅ Confirmed live, 21 Aug** |
| **Issue #3 — Date entry format handling** | **✅ Confirmed live, 20 Aug** |
| Character-gap fix (section name sanitiser, 3 actions) | ✅ Published 22 Aug |
| FA16 defensive guard | ✅ Published 22 Aug (not yet verified by live run) |
| Link-format bug fix | ✅ Published 22 Aug |
| FA33A corruption recovery | ✅ Published 22 Aug |
| FB-05 dated page title fix | ✅ Published and confirmed live 22 Aug |
| Get_items — no structural fault | ✅ Confirmed — transient caching issue on 21 Aug only |
| UJ1–UJ5 core user journeys | Confirmed as of 20 July; UJ2 further confirmed 22 Aug |

## Known intermittent infrastructure issue

**`Send an HTTP request to SharePoint` (mapping row write) intermittently returns 502 BadGateway** after exhausting retries. Not a flow logic fault. Affects some series/occurrences inconsistently. Consequence: if the mapping row isn't written, recapture takes the CREATE path again (potential duplicate page). Workaround: manually insert the mapping row in SharePoint. To be added to the Microsoft support ticket.

---

## Remaining backlog

### High value, not yet started
| Item | Priority | Notes |
|---|---|---|
| `OutStatus` hardcoded to `"OK"` | **Medium-High** | Never differentiates the 6 spec'd values (SUCCESS / RECURRING_SETUP_REQUIRED / PARTIAL_SUCCESS / SETUP_SECTION_NOT_FOUND / SETUP_SECTION_AMBIGUOUS / ERROR). Highest-priority remaining item from the 20 July gap analysis. Recommend Sonnet 4.6, High effort when tackling. |
| UJ3 — stale-row / duplicate-row detection | Medium | Not yet built. |
| UJ4 — section choice, blank-SeriesMasterId fallback, SectionRetryCount | Medium | Not yet built (3 separate gaps). |
| UJ5 — reword/retry option, explicit Stop | Low | Not yet built (2 gaps). |
| FA43 IsRecurring/SeriesMaster coalescing gap | Low | Only Resolved branch wired. Open since July. |

### Published but unverified by live run
| Item | Priority | Notes |
|---|---|---|
| FA16 defensive guard | Low | Belt-and-braces only. Published 22 Aug. Needs a live run to confirm it doesn't break the selection path. |

### Process debt
| Item | Priority | Notes |
|---|---|---|
| Microsoft support ticket | **Overdue** | Drafted but still not submitted. 10+ corruption incidents + intermittent BadGateway. Submit now. |
| Amendment log backfill | Low | `amendment-log.md` process exists but has never been used. |
| `known-good-values-master-reference.md` update | Low | Needs FB-01/02/04/05 expressions added. |
| Duplicate/orphaned test pages in OneNote | Housekeeping | A few stale pages from testing. Worth cleaning up manually. |

### Longer-term / design work
| Item | Priority | Notes |
|---|---|---|
| BadGateway on SharePoint REST write | Medium | Intermittent infrastructure issue on `Send an HTTP request to SharePoint`. Not a flow logic bug. Needs investigation — possibly switch to the native SharePoint connector `Create item` action rather than raw HTTP POST. |
| Flow A solution-aware / VS Code editable | Low | One-time step to make Flow A editable via VS Code rather than Designer-only. Planned but not started. |

---

## Recommended next session focus (Sonnet 4.6)

1. **`OutStatus` differentiation** — High effort. Design the 6-way branching logic carefully before building.
2. **Microsoft support ticket submission** — Standard effort. Overdue.
3. **BadGateway investigation** — consider switching `Send an HTTP request to SharePoint` to a native `Create item` connector action, which may be more resilient.
4. **UJ3 stale-row detection** — if time allows.

## Where to look for detail

- **`session-2026-08-22-afternoon-addendum.md`** — multi-occurrence verification and BadGateway pattern.
- **`session-2026-08-22-backlog-reduction-and-fb04-confirmed.md`** — full morning session.
- **`session-2026-08-21-fb04-build-and-getitems-mystery.md`** — FB-04 build and Get_items investigation.
- **`flow-reference-2026-08-21-full-peek-code-capture.md`** — last full Peek Code + YAML snapshot.

---
*Update at the end of each significant session. If this goes stale, trust the most recent dated session note over this summary.*
