# CURRENT STATE — Teams-OneNote Meeting Capture (start here)

**Last updated:** 31 August 2026 (end of day)
**New Claude instance: read session notes first, most recent first.**

**31 August — Stages 1, 2, 3, and 4 all closed.**
- `session-2026-08-31-stage-3-remove-second-flow-a-call.md` — start here. Stage 3 built and gated. Second Flow A call removed. Cancel fall-through fixed. All six gate tests green.
- `session-2026-08-31-stage-2-and-stage-4.md` — Stage 2 (date in opening utterance + header) and Stage 4 (struck, already complete).
- `session-2026-08-31-stage-1-safety-net.md` — Stage 1 safety net, gated, UJ1-UJ5 regression passed.

**30 August — Stage 0. Four factual checks, no changes.**
- `findings-2026-08-30-stage-0-facts.md`

**29 August — design and review session. No flows changed.**
- `design-2026-08-29-target-state-and-backlog.md` — operative planning document.

**28 August — chat capture scoping.**
- `design-flow-c-chat-transcript-capture.md` — Flow C design, agreed but not built.

**23 August and earlier.**
- `session-2026-08-23-part3-fr01.md`, `session-2026-08-23-part2-fr03-fr02-bug02.md`, `session-2026-08-23-bug01-investigation-and-resolution.md`

---

## TL;DR

**31 August — four stages closed.** Stage 1 (safety net), Stage 2 (date in prompt), Stage 3 (second Flow A call removed), Stage 4 (struck). Flow A, Flow B, and Topic all published.

**Next action: Stage 5 — Perceived latency.** Check `Create_OneNote_Page` connector action against S0.1 first.

**Tomorrow context:** transcript data may arrive. Stage 7 (Flow C) is the relevant stage. Gated on S0.3. Four-section page template in Flow B is a Flow C prerequisite — not yet built.

---

## What's confirmed working

| Item | Status |
|---|---|
| Stage 1 — Safety net (S1W01-S1W05) | Built, gated, regression-tested 31 Aug |
| Stage 2 — Date in opening utterance + candidate list header | Built and gated 31 Aug |
| Stage 3 — Second Flow A call removed, ParseJSON selection | Built and gated 31 Aug |
| Stage 4 — Six OutStatus values in Topic | Already complete 23 Aug, confirmed and struck 31 Aug |
| Cancel fall-through fix (EndDialog) | 31 Aug |
| Out-of-range selection handling (C6F) | 31 Aug |
| BUG-01, BUG-02, FR-01, FR-02, FR-03 | All resolved 23 Aug |
| Issue #1, #2, #3 | All confirmed live |
| FA16, FA43, BadGateway, SeriesMasterId indexing | All confirmed live |

## Remaining backlog

| Stage | Item | Status |
|---|---|---|
| Stage 5 | Perceived latency | Not started — next |
| Stage 6 | Naming convention audit | Not started |
| Stage 7 | Child-flow extraction and Flow C | Not started, gated on S0.3 |

## Process debt

| Item | Priority | Notes |
|---|---|---|
| Microsoft support ticket | Overdue — please submit | `microsoft-discussion-brief-corruption-bug.md` |
| Amendment log | Needs updating | 31 Aug Stage 2 and Stage 3 changes not yet logged |
| `known-good-values-flow-a-reference.md` | Needs Stage 3 additions | FA12B and candidatesjson not yet documented |
| Flow C prerequisite: four-section page template | Not built | Flow B change needed before Flow C writes to structured sections |

## New backlog items from 31 Aug

- **OnlineMeetingUrl extraction from body HTML** — connector returns no `onlineMeeting` object; always `''`. Separate work item.
- **FA29B unguarded substring** — same class of bug as FA12B fix. Add to Stage 6.
- **FA11/FA12 dead code removal** — never evaluated. Remove in Stage 6.

## Open questions

- **Does `Create_OneNote_Page` use the same connector action tested in S0.1?** Gates Stage 5 path.
- **Is S0.3 confirmed for the environment?** Gates Stage 7.
- **Do attendees appear in page content today?**
- **Recurring-chat pagination gap** — open from 28 Aug, close before Flow C is production-ready.
- **Fix 1 partial** — `Filter_Pages_By_Title` inside `Apply_to_each_Existing_Section` still has unguarded `formatDateTime` on `text_5`. Deferred.

## Working-method notes

- `if()` short-circuits in WDL — confirmed 31 Aug scratch test
- `MatchOptions.Contains & MatchOptions.IgnoreCase` — valid Copilot Studio Power Fx
- `ParseJSON` + `Index` + `.FieldName` + `Text()` — confirmed working in Copilot Studio Power Fx
- `Index()` is 1-based in Copilot Studio Power Fx
- Office 365 connector shape is flat — `start` is a plain string, no `start.dateTime`; no `onlineMeeting` object; no `type` field
- `EndDialog` — confirmed valid in Copilot Studio Topic YAML
- `PA - Scratch Diagnostics` — WDL expressions. `ZZ - Scratch ParseJSON Test` — Power Fx / Topic expressions
- No dashes in action names — em-dashes suspected corruption trigger
- All contract changes additive

## Where to look

- `session-2026-08-31-stage-3-remove-second-flow-a-call.md` — Stage 3 detail, errors, gate
- `session-2026-08-31-stage-2-and-stage-4.md` — Stage 2 and Stage 4
- `session-2026-08-31-stage-1-safety-net.md` — Stage 1
- `design-2026-08-29-target-state-and-backlog.md` — operative planning document
- `known-good-values-master-reference.md` — Flow B reference (current as of Stage 1)
- `known-good-values-flow-a-reference.md` — Flow A reference (needs Stage 3 additions)
- `microsoft-discussion-brief-corruption-bug.md` — ready to submit

---
*Update at the end of each significant session.*
