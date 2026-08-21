# Plan for next session — backlog reduction, prioritised by effort

**Written:** 21 August 2026, end of session, for the next working session ("tomorrow").
**Goal:** close out several smaller, well-scoped items and get FB-04 actually verified live — without touching anything structurally risky.

Read `session-2026-08-21-fb04-build-and-getitems-mystery.md` and `CURRENT-STATE.md` first if picking this up fresh.

---

## Suggested order, with model/effort per task

| # | Task | Model / effort | Why |
|---|---|---|---|
| 1 | **Check SharePoint content approval on `RecurringMeetingSectionMap`** (List Settings → Versioning settings) — leading hypothesis for the `Get_items` empty-result issue | Sonnet 5, Standard | Two-click lookup, not reasoning-heavy. Do this first — it unblocks everything else. |
| 1b | *If negative:* work through remaining `Get_items` diagnostic steps (connection reference check, isolated scratch-flow test) | **Switch to Opus 5** for this portion only | Back to open-ended diagnosis with thin evidence — worth the heavier model. Switch back to Sonnet 5 once resolved. |
| 2 | `Compose_SafeSectionName` character-gap fix — extend the existing `replace()` chain to include `\`, `\|`, `#`, `'`, `%`, `~` | Sonnet 5, Standard | Known fix, mechanical, same pattern already used elsewhere in the flow. |
| 3 | FA16 defensive guard — build the digit-strip guard already designed in `fix-2026-08-20-3-datehandling-resolved.md` | Sonnet 5, Standard | Fully scoped already, just needs building. |
| 4 | Link-format bug — swap `PageSelfUrl` for `oneNoteWebUrl` where the flow currently returns the wrong one | Sonnet 5, Standard | Small, low blast radius, cosmetic but quick. |
| 5 | One-off branch title fix — confirm via test (already built, unconfirmed) | Sonnet 5, Standard | Just running and reading a result. |
| 6 | **FB-05** (new, scoped 21 Aug evening) — fix `Compose_SafePageTitle` / `Compose_SafePageTitle_OneOff` to incorporate the dated title (`Topic.PageTitle`), not just the raw meeting title | Sonnet 5, Standard | Straightforward once unblocked by #1; needed before FB-04 can be meaningfully tested on newly-created pages. |
| 7 | Re-run the full #1 test matrix — **start with a genuinely new occurrence date**, not 19 Aug again — to properly verify FB-04 live for the first time | Sonnet 5, Standard | Execution once 1, 6 are done. This is the biggest single win on the board — closes out the last open piece of #1. |
| 8 | `OutStatus` six-way differentiation (SUCCESS/RECURRING_SETUP_REQUIRED/PARTIAL_SUCCESS/SETUP_SECTION_NOT_FOUND/SETUP_SECTION_AMBIGUOUS/ERROR) | Sonnet 5, **High effort** | Not mysterious, but the branching design deserves careful up-front thought so it doesn't become its own future bug. Flagged as highest-priority item in the original 20 July gap analysis — untouched all cycle. |

## Deliberately not in scope for tomorrow (bigger, needs real design work)
- UJ3 stale-row detection
- UJ4: section choice, blank-`SeriesMasterId` fallback, `Topic.SectionRetryCount`
- UJ5: reword/retry, explicit Stop
- FA43's `IsRecurring`/`SeriesMaster` coalescing gap
- Microsoft support ticket submission (process task, not build work — but still overdue, worth doing alongside if time allows)

## Working-method reminders carried forward
- Use the **`PA - Scratch Diagnostics`** standalone flow for any isolated expression testing — proven safe and effective 21 Aug evening, avoids corruption risk from editing live canvases.
- Evidence-first: Peek Code / Activity trace before proposing fixes, always.
- Snapshot immediately before, Flow-check immediately after, any structural edit.
- Batch structural edits in one sitting rather than drip-feeding them.

---
*This plan supersedes nothing — it's additive guidance for sequencing tomorrow's session. If priorities shift, `CURRENT-STATE.md` remains the source of truth for overall project status.*
