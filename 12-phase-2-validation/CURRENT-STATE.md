# CURRENT STATE — Teams-OneNote Meeting Capture (start here)

**Last updated:** 5 September 2026 (evening — corruption recurrence + new blocking defect, unresolved)
**New Claude instance: read session notes first, most recent first.**

**5 September (evening) — BLOCKING: live capture still not working. Two separate issues.**
- `session-handover-2026-09-05-corruption-recurrence-and-oneoff-d2-badrequest.md` — (1) corruption recurred 3x in one session, including a new "value present but empty string" variant, and including re-corruption of already-restored actions — all restored, Flow Checker 0 errors, published. (2) Separate, apparently-genuine defect found on top: `Create_Page_OneOff` fails with BadRequest (`sectionId` invalid/empty) in the one-off fallback branch under `Condition_Is_Genuine_Existing_Page` (false branch). This branch contains two actions (`Set_varPageAction_Created_D2`, `Set_varOutputPageSelfUrl_Created_D2`) not present in `known-good-values-master-reference.md` — likely never exercised end-to-end before tonight. **Next session: start by checking `varTargetSectionPagesUrl`'s actual value at the point of failure in run history** before attempting a fix.

**31 August (evening) — Flow C design decisions settled.**
- Handoff doc (inline) — chat chain proven in scratch. Append-only confirmed. Four-section page template off the table. Internal AI gateway endpoint outstanding. See "Flow C design" section below.

**31 August — Stages 1, 2, 3, and 4 all closed.**
- `session-2026-08-31-stage-3-remove-second-flow-a-call.md` — Stage 3 built and gated. Second Flow A call removed. Cancel fall-through fixed. All six gate tests green.
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

**5 September — BLOCKED.** Corruption recurred and was restored (again — see incident log below), but a live capture still fails on a separate defect: the one-off "not a genuine existing page" fallback branch calls `Create_Page_OneOff` with an empty `sectionId`. This branch looks unexercised/unverified prior to tonight. Fix this before anything else.

**31 August — four stages closed.** Stage 1 (safety net), Stage 2 (date in prompt), Stage 3 (second Flow A call removed), Stage 4 (struck). Flow A, Flow B, and Topic all published.

**Flow C work paused** behind the 5 September blocker — no point building on top of a capture path that doesn't complete yet.

**Blocker for AI step:** Sainsbury's internal AI gateway endpoint and auth not yet confirmed. Ask internal AI/tech team: "What is the HTTP endpoint and auth method for calling Claude Sonnet or Opus from Power Automate?"

---

## Flow C design decisions (settled 31 Aug)

- New separate flow, new Copilot Studio Topic, on-demand only (user triggers after meeting ends)
- Page lookup via SharePoint mapping table (RecurringMeetingSectionMap) — no fresh calendar resolution
- AI judges and summarises in one pass: structured notes if content worth recording, fixed marker ("No additional discussion captured in chat for this session.") if not
- Image-only messages skipped
- Append-only to OneNote — confirmed by S0.2 (Teams Graph HTTP connector cannot reach OneNote endpoints) and S0.1 (native OneNote connector strips HTML wrapper tags at publish time)
- Four-section page template **off the table** — append-only approach makes it unnecessary; Flow B page creation unchanged
- Each chat capture appends a timestamped "Chat Summary" block, same pattern as Flow B's Compose_UpdateHtmlFragment
- AI call will use Sainsbury's internal AI gateway (not public Anthropic API) — endpoint details not yet confirmed
- JoinUrl column added to RecurringMeetingSectionMap SharePoint list ✓

### Chat chain — proven working in PA - Scratch Diagnostics

- JoinUrl → Teams "Get an online meeting" (lookupType: joinWebUrl) → chatInfo.threadId ✓
- threadId → Teams Graph HTTP `me/chats/{threadId}/messages?$top=50&$orderby=createdDateTime desc` → 200, messages returned ✓
- Test meeting: Supply Chain Product Team Meeting, Friday 29 Aug 2026

### Four findings from message data

1. Thread spans all occurrences — date filter required (`greaterOrEquals(ticks(item()?['createdDateTime']), ticks(meetingStartTime))`)
2. `messageType: "unknownFutureValue"` messages are system events — filter to `messageType: "message"` only
3. Image-only messages exist (GIFs etc.) — skip where body content is purely `<img>` tags
4. `@odata.nextLink` present — pagination gap accepted for first build, address later

### Flow C skeleton — build order

1. Get meeting row from SharePoint mapping table by SeriesMasterId + OccurrenceDate
2. SC01 Get Online Meeting (proven)
3. SC02 Compose ThreadId (proven)
4. SC03 Get Chat Messages (proven)
5. Filter: messageType = "message" only
6. Filter: date >= meeting start time
7. Filter: skip image-only messages
8. Compose: format messages as {speaker}: {text} list (stripping HTML)
9. Hardcoded compose: "Chat Summary — [date] [time]\nUpdated by: Meeting Capture Agent\n\n[hardcoded test summary]"
10. OneNote UpdatePageContent — append to existing page using PageSelfUrl from mapping table
11. Gate against Supply Chain Product Team Meeting (Fri 29 Aug)
12. Wire AI step once internal endpoint confirmed
13. Build new Copilot Studio Topic for chat capture trigger

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
| **Blocking** | `Create_Page_OneOff` BadRequest, empty `sectionId`, one-off D2 fallback branch | **New 5 Sep — start next session here** |
| Flow C | Chat capture skeleton | Blocked behind the above |
| Flow C | AI step | Blocked on internal AI gateway endpoint |
| Flow C | Copilot Studio Topic | After skeleton gated |
| Stage 5 | Perceived latency | Not started |
| Stage 6 | Naming convention audit | Not started |

## Process debt

| Item | Priority | Notes |
|---|---|---|
| Microsoft support ticket | Overdue — please submit | `microsoft-discussion-brief-corruption-bug.md` — 5 Sep session adds a strong new data point (same-session escalation incl. re-corruption of already-restored actions, plus a new empty-string value variant) not yet logged in the brief itself |
| Amendment log | Needs updating | 31 Aug Stage 2 and Stage 3 changes, and 5 Sep corruption incidents, not yet logged |
| `known-good-values-flow-a-reference.md` | Needs Stage 3 additions | FA12B and candidatesjson not yet documented |
| `known-good-values-master-reference.md` | Needs D2 actions added | `Set_varPageAction_Created_D2` / `Set_varOutputPageSelfUrl_Created_D2` restored 5 Sep by inference only, not yet confirmed against a working run — do not treat as known-good until the blocking defect above is fixed and verified |

## New backlog items from 31 Aug

- **OnlineMeetingUrl extraction from body HTML** — connector returns no `onlineMeeting` object; always `''`. Separate work item.
- **FA29B unguarded substring** — same class of bug as FA12B fix. Add to Stage 6.
- **FA11/FA12 dead code removal** — never evaluated. Remove in Stage 6.
- **Flow C pagination gap** — `@odata.nextLink` deferred. Close before Flow C is production-ready.

## New items from 5 Sep

- **Designer load-timing hypothesis, unconfirmed** — reopening the flow shows action values as empty for a few seconds before populating. May explain some (not all) of tonight's Flow Checker/Peek Code false-blank reads. Proposed test (reopen, wait 20-30s untouched, then check) not yet run cleanly.
- **`Set_varOutStatus` corruption variant** — value key retained but set to literal `""`, not removed. New pattern for the Microsoft brief.

## Open questions

- **Does `Create_OneNote_Page` use the same connector action tested in S0.1?** Gates Stage 5 path.
- **What is the OneNote UpdatePageContent append action shape?** Need Peek Code on Flow B's existing update action before building Flow C step 10.
- **Sainsbury's internal AI gateway endpoint and auth** — ask internal AI/tech team before wiring AI step.
- **Do attendees appear in page content today?**
- **Fix 1 partial** — `Filter_Pages_By_Title` inside `Apply_to_each_Existing_Section` still has unguarded `formatDateTime` on `text_5`. Deferred.
- **New 5 Sep — is `varTargetSectionPagesUrl` genuinely never set in the D2 fallback branch (design gap), or set then lost (corruption)?** Check run history before fixing.

## Working-method notes

- `if()` short-circuits in WDL — confirmed 31 Aug scratch test
- `MatchOptions.Contains & MatchOptions.IgnoreCase` — valid Copilot Studio Power Fx
- `ParseJSON` + `Index` + `.FieldName` + `Text()` — confirmed working in Copilot Studio Power Fx
- `Index()` is 1-based in Copilot Studio Power Fx
- Office 365 connector shape is flat — `start` is a plain string, no `start.dateTime`; no `onlineMeeting` object; no `type` field
- `EndDialog` — confirmed valid in Copilot Studio Topic YAML
- `PA - Scratch Diagnostics` — WDL expressions. `ZZ - Scratch ParseJSON Test` — Power Fx / Topic expressions
- No dashes in action names — em-dashes suspected corruption trigger (note: several existing OneOff-branch actions already use em-dashes, predating this rule — not yet retrofitted)
- All contract changes additive

## Where to look

- `session-handover-2026-09-05-corruption-recurrence-and-oneoff-d2-badrequest.md` — tonight's corruption incidents and the new blocking defect
- `session-2026-08-31-stage-3-remove-second-flow-a-call.md` — Stage 3 detail, errors, gate
- `session-2026-08-31-stage-2-and-stage-4.md` — Stage 2 and Stage 4
- `session-2026-08-31-stage-1-safety-net.md` — Stage 1
- `design-2026-08-29-target-state-and-backlog.md` — operative planning document
- `design-flow-c-chat-transcript-capture.md` — Flow C design (28 Aug baseline; 31 Aug decisions supersede where they conflict)
- `known-good-values-master-reference.md` — Flow B reference (current as of Stage 1; D2 actions pending confirmation)
- `known-good-values-flow-a-reference.md` — Flow A reference (needs Stage 3 additions)
- `microsoft-discussion-brief-corruption-bug.md` — ready to submit, needs 5 Sep incident added first

---
*Update at the end of each significant session.*
