# CURRENT STATE — Teams-OneNote Meeting Capture (start here)

**Last updated:** 6 September 2026 (evening — Flow B rolled back to 27 August baseline after repeated corruption)
**New Claude instance: read session notes first, most recent first.**

**6 September (evening) — DECISION: Flow B rolled back to 27 August, 17:58 baseline. This is now the working base going forward.**
- Session opened by reviewing the 5 Sep handover and diagnosing the `Create_Page_OneOff` BadRequest (see 5 Sep entry below) — root cause confirmed as a genuine design gap (not corruption): the False branch of `Condition_Is_Genuine_Existing_Page` reads `varTargetSectionPagesUrl` in three places but nothing in that branch ever sets it. A fix was designed (mirror `Condition_Section_Exists_OneOff`'s section-resolve-or-create pattern into this branch, ending in a real `SetVariable`) but **not yet built**.
- Before building it, a fresh 32-action corruption hit occurred from merely *viewing* Designer (no edits made) — on top of the 5 Sep session's three incidents in one sitting. This pattern was judged too unreliable to keep building on.
- **David restored Flow B to the "27 August, 17:58" version via Version History** (Save draft green banner, Publish green banner, Flow Checker 0 errors) and decided to treat it as the new working baseline, rather than continue forward from the unstable 5 Sep state.
- **Full Peek Code of this 27 Aug baseline was reviewed action-by-action and confirmed clean and internally consistent** — matches `known-good-values-master-reference.md` throughout, includes the UJ4b guard (22 Aug), the full OF01-OF10 one-off build (1 Aug), the `Compose_ExistingPageId` bonus fix (1 Aug), and the corrected paren-balanced `Set_varOutStatus` (23 Aug). Does **not** include Stage 1-3 (31 Aug) or any `_D2` actions (5 Sep) — confirmed absent, as expected for this date.
- **Important finding: the `Create_Page_OneOff` sectionId-empty defect is NOT a 5 Sep regression.** It is already present, unchanged, in this 27 Aug baseline — `Condition_Is_Genuine_Existing_Page`'s else branch has never had a section-resolve step for the "genuine existing page check failed" fallback path, since at least 27 August. The two `_D2`-suffixed actions added 5 Sep were bookkeeping layered on top of this pre-existing gap, not its cause. This defect is latent (rarely hit) rather than newly broken, and remains unfixed in the new baseline.
- **Lost by rolling back:** Stage 1 safety net (S1W01-S1W05), Stage 2 (date in opening utterance), Stage 3 (second Flow A call removed) — all from 31 Aug. Not yet decided whether/how these get reapplied to the new baseline.
- **Next session should**: (1) review Flow A and the Topic against this same 27 Aug baseline for the same clean/corruption/completeness check just done for Flow B, (2) then decide priority order between building the `Create_Page_OneOff` section-resolve fix vs. reapplying Stage 1-3.

**5 September (evening) — corruption recurrence + blocking defect diagnosed (superseded by the rollback above — kept for defect history).**
- `session-handover-2026-09-05-corruption-recurrence-and-oneoff-d2-badrequest.md` — three corruption incidents in one session (including a new "value present but empty string" variant, and re-corruption of already-restored actions), all restored, Flow Checker 0 errors, published. Separately: `Create_Page_OneOff` BadRequest (`sectionId` invalid/empty) diagnosed 6 Sep as a genuine design gap, not corruption (see 6 Sep entry above) — this version is no longer the live baseline.

**31 August (evening) — Flow C design decisions settled.**
- Handoff doc (inline) — chat chain proven in scratch. Append-only confirmed. Four-section page template off the table. Internal AI gateway endpoint outstanding. See "Flow C design" section below.

**31 August — Stages 1, 2, 3, and 4 all closed. (Not present in the current 27 Aug baseline — rolled back past this point, see 6 Sep entry above.)**
- `session-2026-08-31-stage-3-remove-second-flow-a-call.md` — Stage 3 built and gated. Second Flow A call removed. Cancel fall-through fixed. All six gate tests green.
- `session-2026-08-31-stage-2-and-stage-4.md` — Stage 2 (date in opening utterance + header) and Stage 4 (struck, already complete).
- `session-2026-08-31-stage-1-safety-net.md` — Stage 1 safety net, gated, UJ1-UJ5 regression passed.

**30 August — Stage 0. Four factual checks, no changes.**
- `findings-2026-08-30-stage-0-facts.md`

**29 August — design and review session. No flows changed.**
- `design-2026-08-29-target-state-and-backlog.md` — operative planning document.

**28 August — chat capture scoping.**
- `design-flow-c-chat-transcript-capture.md` — Flow C design, agreed but not built.

**27 August — Flow B save point, now the working baseline (see 6 Sep entry above). No dedicated session note exists for this date; confirmed via full Peek Code review 6 Sep that it reflects the 23 Aug backlog-clear state with no functional changes since.**

**23 August and earlier.**
- `session-2026-08-23-part3-fr01.md`, `session-2026-08-23-part2-fr03-fr02-bug02.md`, `session-2026-08-23-bug01-investigation-and-resolution.md`

---

## TL;DR

**6 September — ROLLED BACK.** Flow B is now on the 27 August, 17:58 version, chosen deliberately after repeated corruption made the 5 Sep state too unreliable to keep building on. Confirmed clean via full Peek Code review. **This baseline still has the `Create_Page_OneOff` sectionId-empty defect** — it's latent since at least 27 Aug, not a 5 Sep regression, and remains unfixed. Stage 1-3 (31 Aug) are not present in this baseline and need a decision on reapplication.

**Flow A and the Topic have not yet been reviewed against this baseline** — next session should do the same full Peek Code check for both before further build work.

**Flow C work remains paused** — no point building on top of a capture path that doesn't complete yet, and the base flow itself is being re-established.

**Blocker for AI step (unchanged):** Sainsbury's internal AI gateway endpoint and auth not yet confirmed. Ask internal AI/tech team: "What is the HTTP endpoint and auth method for calling Claude Sonnet or Opus from Power Automate?"

---

## Flow C design decisions (settled 31 Aug — status of underlying Flow B work now uncertain post-rollback, see 6 Sep entry)

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

## What's confirmed working (in the current 27 Aug baseline)

| Item | Status |
|---|---|
| OF01-OF10 — one-off mapping lookup + section resolve/create | Built and live-tested 1 Aug, confirmed present and correct in 27 Aug baseline 6 Sep |
| BUG-01, BUG-02, FR-01, FR-02, FR-03 | All resolved 23 Aug, confirmed present in 27 Aug baseline 6 Sep |
| `Compose_ExistingPageId` fix (was blank inputs) | 1 Aug, confirmed present 6 Sep |
| `Set_varOutStatus` paren-balance fix | 23 Aug, confirmed present 6 Sep |
| UJ4b guard on `Filter_Existing_Mapping` | 22 Aug evening, confirmed present 6 Sep |

## Not present in the current 27 Aug baseline (need a decision)

| Item | Landed | Status |
|---|---|---|
| Stage 1 — Safety net (S1W01-S1W05) | 31 Aug | Lost by rollback — reapply or redesign? |
| Stage 2 — Date in opening utterance + candidate list header | 31 Aug | Lost by rollback |
| Stage 3 — Second Flow A call removed, ParseJSON selection | 31 Aug | Lost by rollback |
| Stage 4 — Six OutStatus values in Topic | 23 Aug (confirmed 31 Aug) | Unconfirmed whether present in 27 Aug Topic — **Topic not yet reviewed post-rollback** |
| Cancel fall-through fix (EndDialog) | 31 Aug | Lost by rollback |
| Out-of-range selection handling (C6F) | 31 Aug | Lost by rollback |

## Remaining backlog

| Priority | Item | Status |
|---|---|---|
| **High** | `Create_Page_OneOff` BadRequest, empty `sectionId`, `Condition_Is_Genuine_Existing_Page` false branch | Confirmed 6 Sep: latent since at least 27 Aug, present in new baseline, still unfixed. Fix designed (mirror `Condition_Section_Exists_OneOff`) but not built. |
| **High** | Review Flow A against 27 Aug baseline | Not yet done — do this before further build work |
| **High** | Review Topic against 27 Aug baseline | Not yet done |
| **Medium** | Decide on Stage 1-3 reapplication | Deliberate rebuild vs. accept the loss for now |
| Flow C | Chat capture skeleton | Blocked behind the above |
| Flow C | AI step | Blocked on internal AI gateway endpoint |
| Flow C | Copilot Studio Topic | After skeleton gated |
| Stage 5 | Perceived latency | Not started |
| Stage 6 | Naming convention audit | Not started |

## Process debt

| Item | Priority | Notes |
|---|---|---|
| Microsoft support ticket | Overdue — please submit | `microsoft-discussion-brief-corruption-bug.md` — 5 Sep and 6 Sep sessions both add strong new data points (same-session escalation incl. re-corruption of already-restored actions, a new empty-string value variant, and now a fresh 32-action hit triggered by merely viewing Designer with zero edits made) — none of this yet logged in the brief itself |
| Amendment log | Needs updating | 31 Aug Stage 2/3 changes (now superseded by rollback), and both 5 Sep and 6 Sep corruption incidents, not yet logged |
| `known-good-values-flow-a-reference.md` | Needs Stage 3 additions | Moot if Stage 3 isn't reapplied — revisit after the Stage 1-3 decision above |
| `known-good-values-master-reference.md` | Accurate as of 27 Aug baseline | Confirmed 6 Sep — matches the restored version throughout. Does not yet document the D2/one-off-fallback section-resolve gap as an open defect — worth adding. |

## New backlog items from 31 Aug (status uncertain post-rollback — revisit if Stage 1-3 reapplied)

- **OnlineMeetingUrl extraction from body HTML** — connector returns no `onlineMeeting` object; always `''`. Separate work item.
- **FA29B unguarded substring** — same class of bug as FA12B fix. Add to Stage 6.
- **FA11/FA12 dead code removal** — never evaluated. Remove in Stage 6.
- **Flow C pagination gap** — `@odata.nextLink` deferred. Close before Flow C is production-ready.

## Open questions

- **Does `Create_OneNote_Page` use the same connector action tested in S0.1?** Gates Stage 5 path.
- **What is the OneNote UpdatePageContent append action shape?** Need Peek Code on Flow B's existing update action before building Flow C step 10.
- **Sainsbury's internal AI gateway endpoint and auth** — ask internal AI/tech team before wiring AI step.
- **Do attendees appear in page content today?**
- **Fix 1 partial** — `Filter_Pages_By_Title` inside `Apply_to_each_Existing_Section` still has unguarded `formatDateTime` on `text_5`. Confirmed still present in 27 Aug baseline 6 Sep. Deferred.
- **New 6 Sep — should the `Create_Page_OneOff` section-resolve fix be built against this baseline now, or after Flow A/Topic review is complete?**
- **New 6 Sep — is it worth investigating why Designer merely being opened (no edits) triggered a fresh 32-action corruption hit tonight?** Strongest evidence yet for the Microsoft ticket.

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
- **New 6 Sep — a full Peek Code action-by-action review against `known-good-values-master-reference.md` is an effective way to validate a restored/rolled-back version before trusting it as a baseline.** Worth repeating for Flow A and the Topic.

## Where to look

- `session-handover-2026-09-05-corruption-recurrence-and-oneoff-d2-badrequest.md` — the corruption incidents and defect diagnosis that led to the 6 Sep rollback decision
- `known-good-values-master-reference.md` — Flow B reference, confirmed accurate against the current 27 Aug baseline 6 Sep
- `known-good-values-flow-a-reference.md` — Flow A reference (predates Stage 3 — now consistent with the rolled-back baseline, but not yet re-verified)
- `design-2026-08-29-target-state-and-backlog.md` — operative planning document (predates rollback — Stage 1-4 content now needs a status re-check)
- `design-flow-c-chat-transcript-capture.md` — Flow C design (28 Aug baseline)
- `microsoft-discussion-brief-corruption-bug.md` — ready to submit, needs 5 Sep and 6 Sep incidents added first

---
*Update at the end of each significant session.*
