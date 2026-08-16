# CURRENT STATE — Teams-OneNote Meeting Capture (start here)

**Last updated:** 16 August 2026, end of session
**Purpose:** this is the single page to read before anything else in this folder. Everything below is the current, verified truth about the flow. For full investigation detail on any item, follow the linked handover.

---

## TL;DR

The flow works for the common case. Two real bugs were closed this week (Bug 9, missing page titles). Two things remain genuinely open (a rare edge-case fix and an intermittent timing issue), and a significant platform-behaviour finding (Express mode instability) needs to go to Microsoft. Nothing is currently broken — Flow Checker is clean and the flow is published.

---

## What's confirmed working right now

| Item | Status | Evidence |
|---|---|---|
| Existing-page recapture (Bug 9) | **Closed**, workaround in place | `handover-2026-08-16-bug9-closed-workaround-confirmed.md` |
| New page titles — recurring meetings | **Fixed and confirmed** | `handover-2026-08-16-page-title-fix-recurring-confirmed.md` |
| New page titles — ordinary one-off first captures | **Already covered** by the recurring-branch fix above (both paths share `Create_OneNote_Page` for first-time captures) | `handover-2026-08-16-oneoff-title-fix-built-unconfirmed.md` |
| UJ1–UJ5 (core user journeys) | Confirmed as of 20 July; not re-verified since | `uj1`–`uj5-validation-record.md` |
| Bug 7 (recurring second-capture) | Fixed, confirmed | `handover-2026-08-08-bug7-recurring-second-capture-sectionid-mismatch.md` |
| GitHub MCP write access | Working reliably | see `AMEND` log / June handovers |

## What's still open

| Item | Priority | Notes |
|---|---|---|
| Recurring title-set intermittent `404` race | Medium | `Set_PageTitle_Recurring` occasionally fails even against a freshly-verified page ID. Delay-based fix attempted and abandoned (see Express mode issue below). **Recommended next approach: `Do until` retry/poll instead of a Delay action.** `handover-2026-08-16-page-title-fix-recurring-confirmed.md` |
| One-off branch title fix (`Create_Page_OneOff`) | Low | Built, Flow Checker clean, published — but genuinely unconfirmed. Only reachable via a rare stale-mapping edge case, not ordinary captures. `handover-2026-08-16-oneoff-title-fix-built-unconfirmed.md` |
| Bug 9 workaround → real fix | Medium | Currently takes "the section's first page" as a stopgap. Will break once a section legitimately holds 2+ pages. Should be reverted to genuine title-matching once titles are trustworthy everywhere (needs the one-off fix confirmed first). |
| Tail-section anomaly | Low-Medium | `Compose SP Item Count` → `Respond to the agent` showed "Not specified"/unreached status in one run this session, cause not diagnosed. Not reproduced or ruled in/out on a clean retest. |
| Notebook / SharePoint test-data cleanup | Low | Mostly done 16 August; ongoing housekeeping. `2026-08-16-test-data-cleanup-note.md` |

## Known platform issues (not flow bugs — flag to Microsoft)

1. **Mass value-blanking corruption** — a recurring pattern all week where ~26 `SetVariable` actions lose their `value` field simultaneously. Multiple trigger types now confirmed: Designer canvas edits, publish events, and (new 16 August) **flow-level settings changes**.
2. **Express mode will not stay off** — toggled off and saved explicitly, twice, and reverted to Enabled on its own both times, each time triggering the 26-action corruption above. This is the most operationally serious finding of the week — not mitigated by careful editing discipline, since no Designer edit was involved. See `handover-2026-08-16-session-close-express-mode-unstable.md`.
3. Various smaller quirks (publish-only validation gaps, self-resolving missing fields, BadGateway masking real success) — see `MICROSOFT-SUPPORT-TICKET-DRAFT-2026-08-15.md` for the full catalogue.

**The Microsoft ticket is drafted but not yet submitted.** David plans to submit next week. Before submitting, fold in the 16 August Express-mode findings (currently the strongest, most reproducible evidence in the whole investigation).

## Where to look for detail

- **`amendment-log.md`** — formal, numbered list of every confirmed fix, in order. Best single source for "what changed and why."
- **`flow-reference-2026-08-16-pre-page-title-fix-backup.md`** — full Peek Code snapshot of the known-good flow state as of this morning. Useful restore reference if corruption strikes again.
- **Dated `handover-*.md` files** — full investigation narrative for anything above. Filenames describe their content; most recent (16 August) are the most relevant for current work.
- **`living-audit.md` / `living-audit-topic.md`** — older running audit docs; per earlier notes, entries from before ~25 June may be stale and should be re-verified rather than trusted outright.

## Before your presentation

Suggested framing: two real bugs found and closed this week (with evidence), one significant platform-reliability finding ready to escalate to Microsoft, and a clear, prioritised list of what's left — nothing currently broken. The repo has full documentation and an established recovery process that's already proven itself twice.

---

*This file should be updated at the end of each significant session. If it goes stale, trust the most recent dated handover over this summary.*
