# CURRENT STATE — Teams-OneNote Meeting Capture (start here)

**Last updated:** 21 August 2026, end of day
**⚠️ If you are a new Claude instance picking this up, read `HANDOVER-2026-08-21.md` first** — it's written specifically for a fresh session with no prior context, and is more actionable than this summary alone.

**Purpose:** this is the single page to read before anything else in this folder. Everything below is the current, verified truth about the flow. For full investigation detail on any item, follow the linked handover.

---

## TL;DR

Three field-reported issues were investigated 20–21 August. **#2 (recapture content loss) and #3 (date entry format) are fully fixed and confirmed live.** **#1 (per-occurrence recurring pages) has its foundation built and confirmed live, with three of four remaining sub-changes built and saved in draft — the final piece (FB-04, replacing the Bug 9 workaround with real date-matching) is not yet built.** Today's session (21 Aug) was cut short by repeated, worsening platform corruption incidents — deliberately paused rather than pushed through, to avoid losing track of what's genuinely fixed vs. silently broken.

---

## What's confirmed working right now

| Item | Status | Evidence |
|---|---|---|
| **Date entry — loose formats + slash dates (#3)** | **Fixed and confirmed live, 20 Aug** | `fix-2026-08-20-3-datehandling-resolved.md` |
| **Recapture content loss (#2)** | **Fixed and confirmed live, 21 Aug** | `fix-2026-08-21-2-appendcontent-resolved.md` |
| **#1 foundation — `OccurrenceDate` field flowing end-to-end** | **Published and confirmed live, 21 Aug** | `session-2026-08-21-fb-progress-and-incidents.md` |
| Existing-page recapture (Bug 9) | Closed, workaround in place — **this workaround is the hard blocker for #1's final piece (FB-04)** | `handover-2026-08-16-bug9-closed-workaround-confirmed.md` |
| New page titles — recurring meetings | Fixed and confirmed | `handover-2026-08-16-page-title-fix-recurring-confirmed.md` |
| UJ1–UJ5 (core user journeys) | Confirmed as of 20 July; not re-verified since | `uj1`–`uj5-validation-record.md` |
| Bug 7 (recurring second-capture) | Fixed, confirmed | `handover-2026-08-08-bug7-recurring-second-capture-sectionid-mismatch.md` |
| GitHub MCP write access | Working reliably | see `AMEND` log / June handovers |

## What's still open

| Item | Priority | Notes |
|---|---|---|
| **#1 — Ref FB-04 (final piece)** | **Ready to build, session paused before starting** | Replace `Compose_RealExistingPageId` (Bug 9 workaround) with genuine date-based page matching. Full detail and exact expressions in `session-2026-08-21-fb-progress-and-incidents.md` and `HANDOVER-2026-08-21.md`. **One thing to verify first**: whether `formatDateTime(text_5, 'd MMM yyyy')` produces the exact display format used in page titles — this was never confirmed live due to repeated corruption interruptions. |
| **#1 — Flow B publish** | Pending | FB-01 (matching filter) and FB-02 (mapping write) are built and saved in draft, confirmed via Peek Code, but **not yet published**. Bundle with FB-04 for one combined publish + full test. |
| **#1 — Topic publish** | Pending | FB-03 (uniform page title) is saved in Topic draft, **not yet published**. Publish together with Flow B once FB-04 is done. |
| FA16 defensive guard (Flow A) | Low | Belt-and-braces only. Expression ready in `fix-2026-08-20-3-datehandling-resolved.md`, not yet built. |
| Recurring title-set intermittent `404` race | Medium | `handover-2026-08-16-page-title-fix-recurring-confirmed.md` |
| One-off branch title fix (`Create_Page_OneOff`) | Low | Built, unconfirmed, rare edge case. |
| Topic YAML re-export to repo | Low (housekeeping) | Committed `topic-export-2026-07-31.yaml` is now significantly stale (predates #3 fix, C6D, `text_5` binding, FB-03). Re-export next calm session. |
| Link-format bug (`PageSelfUrl` vs `oneNoteWebUrl`) | Medium | Still present as of 21 Aug testing. Logged since 6/8 Aug, not yet fixed. |
| `Compose_SafeSectionName` character gap | Low-Medium | Missing `\`, `|`, `#`, `'`, `%`, `~` from its sanitiser strip list (vs. OneNote's full forbidden-character set). Surfaced 21 Aug via a `Create_Section_Recurring` BadRequest, but the specific title that failed ("121 Simon / David") didn't actually contain any of the missing characters — root cause not fully confirmed, may have been a transient corruption hit instead. Worth a fresh, calm look. |

## Known platform issues (not flow bugs — flag to Microsoft)

1. **Mass value-blanking corruption** — a recurring pattern where `SetVariable`/Compose actions lose their `value` field. **9+ incidents logged as of 21 August**, including two same-day repeats on the identical action (`OF05c`) during the #1 build session, and a new observed correlation: **structural canvas edits (adding/moving actions) appear to trigger it more reliably than pure expression edits.** See `session-2026-08-21-fb-progress-and-incidents.md` for the latest incidents and `handover-2026-08-20-bug8-resolved-corruption-rootcause.md` for the earlier pattern analysis.
2. **Express mode will not stay off** — see `handover-2026-08-16-session-close-express-mode-unstable.md`.
3. Various smaller quirks — see `MICROSOFT-SUPPORT-TICKET-DRAFT-2026-08-15.md`.

**The Microsoft ticket is drafted but STILL not submitted, despite mounting evidence.** This should now be treated as a priority action, not just a housekeeping item — fold in all incidents through 21 August before submitting.

## Where to look for detail

- **`HANDOVER-2026-08-21.md`** — written for a fresh Claude session with zero prior context. Start here if you're new to this project.
- **`amendment-log.md`** — formal, numbered list of every confirmed fix, in order.
- **`known-good-values-master-reference.md`** — restore-focused reference for Flow B's SetVariable/Compose expressions. **Now used and proven multiple times** as the fast-recovery mechanism during corruption incidents. Does not yet reflect FB-01/FB-02's new expressions — update alongside FB-04 completion.
- **`flow-reference-2026-08-20-full-peek-code-capture.md`** — Peek Code snapshot as of 20 August. Several entries are now stale (Compose_UpdateHtmlFragment, Filter_Existing_Mapping, the mapping-write body) — treat as historical, not current.
- **`design-amendment-2026-08-20-per-occurrence-recurring-pages.md`** — the original #1 design.
- **`session-2026-08-21-fb-progress-and-incidents.md`** — today's detailed build log and incident record.
- **Dated `handover-*.md` and `fix-*.md` files** — full investigation narrative for anything above.

## Before your presentation

Suggested framing: three field-reported issues investigated with an evidence-first approach; two fully fixed and confirmed live; the third has a tested, live-confirmed foundation with the final piece scoped and ready. Multiple real platform bugs closed. A significant, now well-evidenced platform-reliability finding ready to escalate to Microsoft — this alone may be worth surfacing prominently, given how much time it has cost across sessions.

---

*This file should be updated at the end of each significant session. If it goes stale, trust the most recent dated handover over this summary.*
