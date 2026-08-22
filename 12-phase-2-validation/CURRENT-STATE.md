# CURRENT STATE — Teams-OneNote Meeting Capture (start here)

**Last updated:** 22 August 2026, end of session
**⚠️ If you are a new Claude instance picking this up, read `session-2026-08-22-backlog-reduction-and-fb04-confirmed.md` first** — it covers the full context of what was confirmed today.

**Purpose:** this is the single page to read before anything else in this folder.

---

## TL;DR

**Issue #1 (per-occurrence recurring pages) is now fully confirmed live end-to-end.** All four sub-changes (FB-01 through FB-04) plus the companion FB-05 fix are published and verified. The full create → recapture cycle was tested on the 121 Simon / David series (16 Sep 2026 occurrence) and passed completely: new page created with dated title, recapture correctly identified the existing page by date and appended the update fragment without creating a duplicate or discarding existing content.

Issues #2 and #3 remain confirmed from 20-21 Aug.

Remaining work is primarily around `OutStatus` differentiation, UJ3-UJ5 gaps, and process debt. None of these are blockers for normal usage.

---

## What's confirmed working right now

| Item | Status | Evidence |
|---|---|---|
| **Issue #1 — Per-occurrence recurring pages (FB-01 through FB-05)** | **✅ Fully confirmed live, 22 Aug** | `session-2026-08-22-backlog-reduction-and-fb04-confirmed.md` |
| **Issue #2 — Recapture content protection** | **✅ Confirmed live, 21 Aug** | Working alongside FB-04 in today's test |
| **Issue #3 — Date entry format handling** | **✅ Confirmed live, 20 Aug** | `fix-2026-08-20-3-datehandling-resolved.md` |
| Character-gap fix (section name sanitiser, 3 actions) | ✅ Published 22 Aug | |
| FA16 defensive guard | ✅ Published 22 Aug | Not yet verified by live run |
| Link-format bug fix | ✅ Published 22 Aug | `PageWebUrl` confirmed populated in mapping rows |
| FA33A corruption recovery | ✅ Published 22 Aug | |
| Get_items — no structural fault | ✅ Confirmed 22 Aug | Transient caching issue on 21 Aug; returns correct data on subsequent calls |
| UJ1–UJ5 (core user journeys) | Confirmed as of 20 July; UJ2 further confirmed today | |

## What's still open

| Item | Priority | Notes |
|---|---|---|
| `OutStatus` hardcoded to `"OK"` | **Medium** | Never differentiates the 6 spec'd values. Highest-priority remaining gap from the 20 July gap analysis. Recommend tackling next. |
| FA16 live verification | Low | Belt-and-braces guard published but not yet run. |
| UJ3 stale-row detection | Medium | Not yet built. |
| UJ4 gaps (section choice, blank-SeriesMasterId fallback, SectionRetryCount) | Medium | Not yet built. |
| UJ5 gaps (reword/retry, explicit Stop) | Low | Not yet built. |
| FA43 IsRecurring/SeriesMaster coalescing gap | Low | Open since July. |
| Amendment log backfill | Low | Process debt. |
| `known-good-values-master-reference.md` update | Low | Needs FB-01/02/04/05 expressions added. |
| Microsoft support ticket | **Overdue** | Still not submitted. 10+ corruption incidents. |
| Duplicate/orphaned test pages in OneNote | Housekeeping | A few stale pages from testing (19 Aug x2, 22 Aug x1 failed run). Worth cleaning up manually. |

## Known platform issues

1. **Mass value-blanking corruption** — 10+ incidents. Largest was 22 actions on 21 Aug. Correlates with structural canvas edits. Microsoft ticket drafted but not submitted.
2. **Express mode instability** — see `handover-2026-08-16-session-close-express-mode-unstable.md`.
3. **Connection initialisation on first run** — new flows or connections sometimes hit BadGateway on first execution, resolving on retry. Not a flow bug.

## Recommended next session focus

1. **`OutStatus` differentiation** — Sonnet 4.6, High effort. Design the 6-way branching logic carefully before building.
2. **Microsoft support ticket submission** — overdue, process task.
3. **UJ3 stale-row detection** — if time allows after OutStatus.

## Where to look for detail

- **`session-2026-08-22-backlog-reduction-and-fb04-confirmed.md`** — most recent, covers today's full session including FB-04 live confirmation.
- **`session-2026-08-21-fb04-build-and-getitems-mystery.md`** — FB-04 build, Get_items investigation.
- **`session-2026-08-21-driftcheck-and-corruption-incident.md`** — 22-action corruption recovery.
- **`flow-reference-2026-08-21-full-peek-code-capture.md`** — last full Peek Code + YAML snapshot (21 Aug evening, pre-today's changes).
- **`plan-2026-08-22-backlog-reduction.md`** — today's plan (now substantially complete).

---
*Update at the end of each significant session. If this goes stale, trust the most recent dated session note over this summary.*
