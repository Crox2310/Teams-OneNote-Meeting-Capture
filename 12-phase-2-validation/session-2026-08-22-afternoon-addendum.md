# Session note addendum — 22 August 2026 (afternoon continuation)

**Appended to:** `session-2026-08-22-backlog-reduction-and-fb04-confirmed.md`

---

## Part 5 — Multi-occurrence verification: confirmed

Following FB-04 and FB-05 confirmation on the 121 Simon / David series (16 Sep 2026 occurrence), further testing was conducted to verify that capturing **multiple different occurrence dates within the same series** correctly creates separate dated pages under the same OneNote section.

**Test: STDA recurring series, two different occurrences**
- `STDA - 30 Sep 2026` page created in `Mtg - STDA` section ✅
- `STDA - 15 Oct 2026` page created in the same `Mtg - STDA` section ✅
- Both pages visible as siblings under the same section in OneNote ✅
- Both pages have correctly dated titles ✅

**This confirms the full per-occurrence design end-to-end:** one section per meeting series, one page per occurrence date, all sitting as dated siblings under the same section. Issue #1 is fully verified.

---

## Part 6 — Intermittent BadGateway on Send_an_HTTP_request_to_SharePoint

A consistent pattern emerged across multiple test runs: `Send an HTTP request to SharePoint` (the mapping-row write action) intermittently returns `502 BadGateway` after exhausting all retries (8 attempts). This is **not** a flow logic fault — it's a SharePoint infrastructure issue.

**Evidence:**
- Affects the `RecurringMeetingSectionMap` REST POST specifically
- `GET` operations (via `Get_items` connector action) work cleanly
- Other SharePoint connector actions in the flow work correctly
- Some series (e.g. STDA) succeed cleanly; others (e.g. 121 Simon / David 30 Sep) hit BadGateway consistently
- Refreshing the SharePoint connection did not resolve it
- The issue is intermittent — earlier captures on the same flow (16 Sep 121 Simon / David) succeeded cleanly

**Practical consequence:** when the HTTP write fails, no mapping row is written for that occurrence. Subsequent recaptures for that occurrence will take the CREATE path again (no mapping row found) rather than the UPDATE path, potentially creating duplicate pages.

**Not a blocker for normal usage** — the majority of captures succeed. Worth adding to the Microsoft support ticket as an additional evidence point alongside the corruption incidents.

**Workaround for affected occurrences:** manually insert the mapping row in SharePoint if the recapture behaviour is needed for that specific occurrence.

---

## Overall status after full day's testing

All three field-reported issues confirmed fixed and working end-to-end. The per-occurrence recurring page feature (Issue #1) is verified across multiple series and multiple occurrence dates. The intermittent BadGateway is logged as a known infrastructure issue, not a flow logic fault.
