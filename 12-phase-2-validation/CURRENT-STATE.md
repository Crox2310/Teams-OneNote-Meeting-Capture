# CURRENT STATE — Teams-OneNote Meeting Capture (start here)

**Last updated:** 21 August 2026, morning session
**Purpose:** this is the single page to read before anything else in this folder. Everything below is the current, verified truth about the flow. For full investigation detail on any item, follow the linked handover.

---

## TL;DR

Three field-reported issues (per-occurrence recurring pages, recapture content loss, date entry format) were investigated 20–21 August. **Two are now fixed and confirmed live: date handling (#3) and recapture content loss (#2).** The third (#1, per-occurrence recurring pages) has a complete, evidence-backed design and is ready to build — its dependency on #2 being fixed first is now satisfied. Flow Checker is clean and the flow is published.

---

## What's confirmed working right now

| Item | Status | Evidence |
|---|---|---|
| **Date entry — loose formats + slash dates (#3)** | **Fixed and confirmed live, 20 Aug** | `fix-2026-08-20-3-datehandling-resolved.md` |
| **Recapture content loss (#2)** | **Fixed and confirmed live, 21 Aug** | `fix-2026-08-21-2-appendcontent-resolved.md` |
| Existing-page recapture (Bug 9) | Closed, workaround in place — **note: this workaround is now the hard blocker for #1, see below** | `handover-2026-08-16-bug9-closed-workaround-confirmed.md` |
| New page titles — recurring meetings | Fixed and confirmed | `handover-2026-08-16-page-title-fix-recurring-confirmed.md` |
| New page titles — ordinary one-off first captures | Already covered by the recurring-branch fix above | `handover-2026-08-16-oneoff-title-fix-built-unconfirmed.md` |
| UJ1–UJ5 (core user journeys) | Confirmed as of 20 July; not re-verified since | `uj1`–`uj5-validation-record.md` |
| Bug 7 (recurring second-capture) | Fixed, confirmed | `handover-2026-08-08-bug7-recurring-second-capture-sectionid-mismatch.md` |
| GitHub MCP write access | Working reliably | see `AMEND` log / June handovers |

## What's still open

| Item | Priority | Notes |
|---|---|---|
| **Per-occurrence recurring pages (#1)** | **Ready to build** | Design complete and evidence-backed. `design-amendment-2026-08-20-per-occurrence-recurring-pages.md`. Its dependency on #2 (content loss) is now satisfied — build can proceed. Requires replacing the Bug 9 "first page in section" workaround with genuine date-based matching (see row above). |
| FA16 defensive guard (Flow A) | Low | Belt-and-braces only — the Topic-side #3 fix already prevents date text reaching Flow A. Digit-strip guard expression ready in `fix-2026-08-20-3-datehandling-resolved.md`, not yet built. |
| Recurring title-set intermittent `404` race | Medium | `Set_PageTitle_Recurring` occasionally fails even against a freshly-verified page ID. **Recommended approach: `Do until` retry/poll instead of a Delay action.** `handover-2026-08-16-page-title-fix-recurring-confirmed.md` |
| One-off branch title fix (`Create_Page_OneOff`) | Low | Built, Flow Checker clean, published — genuinely unconfirmed. Rare edge case only. `handover-2026-08-16-oneoff-title-fix-built-unconfirmed.md` |
| Tail-section anomaly | Low-Medium | One unreproduced anomaly from a prior session, not diagnosed. |
| Notebook / SharePoint test-data cleanup | Low | Ongoing housekeeping. `2026-08-16-test-data-cleanup-note.md` |
| Topic YAML re-export to repo | Low (housekeeping) | The live Topic now has the #3 fix (C6C slash+text parsing, C6D number-selection, else re-prompt) but the committed `topic-export-2026-07-31.yaml` predates it. Re-export and commit next time convenient. |
| Link-format bug (`PageSelfUrl` vs `oneNoteWebUrl`) | Medium | Still present as of 21 Aug testing — clicking some returned page links gives `C40001` auth error. Logged since 6/8 Aug, not yet fixed. |

## Known platform issues (not flow bugs — flag to Microsoft)

1. **Mass value-blanking corruption** — a recurring pattern where ~26 `SetVariable` actions lose their `value` field simultaneously. Multiple trigger types confirmed: Designer canvas edits, publish events, flow-level settings changes. 7 incidents logged as of 20 August; one instance (Bug 8) was found to have masqueraded as a distinct logic bug — see `handover-2026-08-20-bug8-resolved-corruption-rootcause.md`.
2. **Express mode will not stay off** — toggled off and saved explicitly, twice, and reverted to Enabled on its own both times, each time triggering the corruption above. See `handover-2026-08-16-session-close-express-mode-unstable.md`.
3. Various smaller quirks — see `MICROSOFT-SUPPORT-TICKET-DRAFT-2026-08-15.md` for the full catalogue.

**The Microsoft ticket is drafted but not yet submitted.** Fold in the Express-mode findings and the "corruption masquerading as a logic bug" finding (Bug 8) before submitting.

## Where to look for detail

- **`amendment-log.md`** — formal, numbered list of every confirmed fix, in order.
- **`known-good-values-master-reference.md`** — restore-focused reference of every SetVariable/Compose expression in Flow B, for fast recovery after a corruption incident. Note: does not yet reflect the #2 fix's new `Compose_UpdateHtmlFragment` expression — update this alongside next session.
- **`flow-reference-2026-08-20-full-peek-code-capture.md`** — full Peek Code snapshot of Flow B as of 20 August (pre-#2 fix; the `Compose_UpdateHtmlFragment` entry is now stale, everything else current).
- **Dated `handover-*.md` and `fix-*.md` files** — full investigation narrative for anything above. Most recent (20–21 August) are most relevant for current work.
- **`living-audit.md` / `living-audit-topic.md`** — older running audit docs; entries from before ~25 June may be stale.

## Before your presentation

Suggested framing: three field-reported issues investigated with an evidence-first approach (throwaway diagnostics before any live change); two fully fixed and confirmed live within 24 hours; the third has a complete, ready-to-build design. Two real platform bugs closed this week, one significant platform-reliability finding ready to escalate to Microsoft. The repo has full documentation and an established recovery process proven multiple times.

---

*This file should be updated at the end of each significant session. If it goes stale, trust the most recent dated handover over this summary.*
