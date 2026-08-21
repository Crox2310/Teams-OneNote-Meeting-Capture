# CURRENT STATE — Teams-OneNote Meeting Capture (start here)

**Last updated:** 21 August 2026, end of day (evening session)
**⚠️ If you are a new Claude instance picking this up, read the two most recent session notes first**: `session-2026-08-21-driftcheck-and-corruption-incident.md` (drift check + major corruption recovery) and `session-2026-08-21-fb04-build-and-getitems-mystery.md` (FB-04 build, publish, and a new unresolved `Get_items` issue). Together they're more actionable than this summary alone.

**Purpose:** this is the single page to read before anything else in this folder. Everything below is the current, verified truth about the flow. For full investigation detail on any item, follow the linked handover.

---

## TL;DR

Three field-reported issues were investigated 20–21 August. **#2 (recapture content loss) and #3 (date entry format) are fully fixed and confirmed live.** **#1 (per-occurrence recurring pages)**: all four sub-changes (FB-01 through FB-04) are now **built, Peek-Code-verified, and published**, but **FB-04 has not yet been exercised by any live run** — a same-day test was derailed by a newly discovered, separate, unresolved issue: `Get_items` is returning an empty result set from the `RecurringMeetingSectionMap` SharePoint list despite the list demonstrably containing data. Root cause unknown; a stale/wrong list-ID reference has been ruled out. This is now the practical blocker for further #1 testing. A second new finding: FB-03's dated page title never reaches the actual OneNote page title metadata (only embedded in the page body's HTML), which will separately block FB-04's title-matching logic even once `Get_items` is resolved — recommend scoping as FB-05.

---

## What's confirmed working right now

| Item | Status | Evidence |
|---|---|---|
| **Date entry — loose formats + slash dates (#3)** | **Fixed and confirmed live, 20 Aug** | `fix-2026-08-20-3-datehandling-resolved.md` |
| **Recapture content loss (#2)** | **Fixed and confirmed live, 21 Aug** | `fix-2026-08-21-2-appendcontent-resolved.md` |
| **#1 foundation — `OccurrenceDate` field flowing end-to-end** | **Published and confirmed live, 21 Aug** | `session-2026-08-21-fb-progress-and-incidents.md` |
| **#1 — FB-01, FB-02, FB-03, FB-04 (all four sub-changes)** | **Built, Peek-Code-verified, published** — but **FB-04 not yet exercised live** | `session-2026-08-21-fb04-build-and-getitems-mystery.md` |
| `formatDateTime(text_5, 'd MMM yyyy')` format assumption | **Confirmed via isolated scratch-flow test**: outputs `"19 Aug 2026"` | `session-2026-08-21-fb04-build-and-getitems-mystery.md`, Part 1 |
| Existing-page recapture (Bug 9) | Closed, workaround **replaced by FB-04** (pending live verification) | see above |
| New page titles — recurring meetings | Fixed and confirmed (title string only — **does not include date**, see open items) | `handover-2026-08-16-page-title-fix-recurring-confirmed.md` |
| UJ1–UJ5 (core user journeys) | Confirmed as of 20 July; not re-verified since | `uj1`–`uj5-validation-record.md` |
| Bug 7 (recurring second-capture) | Fixed, confirmed | `handover-2026-08-08-bug7-recurring-second-capture-sectionid-mismatch.md` |
| GitHub MCP write access | Working reliably | see `AMEND` log / June handovers |

## What's still open

| Item | Priority | Notes |
|---|---|---|
| **`Get_items` returns empty against a non-empty SharePoint list** | **HIGH — new, blocks all further #1 testing** | Confirmed NOT a stale/wrong list-ID GUID (checked and matches). Root cause unknown. See `session-2026-08-21-fb04-build-and-getitems-mystery.md`, Part 4, for full evidence and recommended next steps (check connection reference, isolate via scratch flow, check for content-approval on the list). |
| **FB-04 not yet verified end-to-end** | **HIGH** | Code confirmed correct via Peek Code diff, but no live run has reached the branch containing it yet — blocked by the `Get_items` issue above. |
| **New: FB-03's dated title never reaches actual OneNote page title metadata** | **HIGH — recommend scoping as FB-05** | `Compose_SafePageTitle` / `Compose_SafePageTitle_OneOff` only use the raw meeting title, never `Topic.PageTitle`'s date-suffixed version. The date only appears inside the page body's HTML `<title>` tag, not as real page title metadata. This will block FB-04's `contains()` title-matching on any *newly created* page even once `Get_items` is fixed. See `session-2026-08-21-fb04-build-and-getitems-mystery.md`, Part 5. |
| FA16 defensive guard (Flow A) | Low | Belt-and-braces only. Expression ready in `fix-2026-08-20-3-datehandling-resolved.md`, not yet built. |
| Recurring title-set intermittent `404` race | Medium | `handover-2026-08-16-page-title-fix-recurring-confirmed.md` |
| One-off branch title fix (`Create_Page_OneOff`) | Low | Built, unconfirmed, rare edge case. |
| Topic YAML re-export to repo | **Done 21 Aug (evening)** | Current full export captured in `flow-reference-2026-08-21-full-peek-code-capture.md`. |
| Link-format bug (`PageSelfUrl` vs `oneNoteWebUrl`) | Medium | Still present as of 21 Aug testing. Logged since 6/8 Aug, not yet fixed. |
| `Compose_SafeSectionName` character gap | Low-Medium | Missing `\`, `|`, `#`, `'`, `%`, `~` from its sanitiser strip list. Root cause not fully confirmed, may have been a transient corruption hit. Worth a fresh, calm look. |
| `varOutStatus` hardcoded to `"OK"` | Medium (pre-existing, longstanding) | Confirmed still present in 21 Aug evening Peek Code capture — `Set_varOutStatus` unconditionally sets `"OK"` regardless of actual outcome. Not touched this session. |

## Known platform issues (not flow bugs — flag to Microsoft)

1. **Mass value-blanking corruption** — a recurring pattern where `SetVariable`/Compose actions lose their `value` field. **10+ incidents logged as of 21 August**, including a new record: a **22-action, dual-branch incident** on 21 Aug afternoon (largest and most broadly distributed to date — see `session-2026-08-21-driftcheck-and-corruption-incident.md`), on top of the prior pattern of repeats on `OF05c` specifically. Continues to correlate with structural canvas edits (adding/moving actions) more than pure expression edits.
2. **Express mode will not stay off** — see `handover-2026-08-16-session-close-express-mode-unstable.md`.
3. Various smaller quirks — see `MICROSOFT-SUPPORT-TICKET-DRAFT-2026-08-15.md`.

**The Microsoft ticket is drafted but STILL not submitted, despite mounting evidence — now including the largest incident logged to date.** This should be treated as overdue, not just a housekeeping item.

## Working-method addition from 21 Aug (evening) — worth carrying forward

A **standalone scratch flow** (`PA - Scratch Diagnostics`) was created this session specifically for testing isolated expressions (e.g. `formatDateTime` behaviour) without touching Flow B's live canvas. This worked cleanly and avoided the corruption risk associated with structural edits to production flows. **Recommend using this pattern by default for any future isolated expression testing**, rather than adding throwaway actions directly into Flow B or the Topic.

## Where to look for detail

- **`session-2026-08-21-fb04-build-and-getitems-mystery.md`** — most recent session: FB-04 build/verification, publish, first live test attempt, and the new `Get_items` + page-title-metadata findings. **Start here for the most current picture.**
- **`session-2026-08-21-driftcheck-and-corruption-incident.md`** — same-day, earlier session: drift check (passed clean) and the 22-action corruption incident + recovery.
- **`flow-reference-2026-08-21-full-peek-code-capture.md`** — current full Peek Code (Flow B) and YAML (Topic) snapshot, captured 21 Aug evening. Supersedes the 20 Aug capture.
- **`HANDOVER-2026-08-21.md`** — original narrative handover written at the start of the day; still useful for background on the three field-reported issues, but treat the two session notes above as more current for #1's status.
- **`amendment-log.md`** — formal, numbered list of every confirmed fix, in order. **Not yet updated with FB-01–FB-04 or today's findings — due for an update.**
- **`known-good-values-master-reference.md`** — restore-focused reference for Flow B's SetVariable/Compose expressions. **Not yet updated with FB-01/FB-02/FB-04's new expressions** — update alongside `Get_items` resolution.
- **`design-amendment-2026-08-20-per-occurrence-recurring-pages.md`** — the original #1 design.
- **Dated `handover-*.md` and `fix-*.md` files** — full investigation narrative for anything above.

## Before your presentation

Suggested framing: three field-reported issues investigated with an evidence-first approach; two fully fixed and confirmed live; the third (#1) has all four planned code changes built, verified, and published, with two newly surfaced blockers standing between "built" and "proven" — both well-understood and scoped, neither a mystery at the code level. Multiple real platform bugs closed, plus a significant, now very well-evidenced platform-reliability finding (including today's largest-yet incident) ready to escalate to Microsoft.

---

*This file should be updated at the end of each significant session. If it goes stale, trust the most recent dated session note over this summary.*
