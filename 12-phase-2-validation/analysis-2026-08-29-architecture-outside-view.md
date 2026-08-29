# Architecture review — outside view, 29 August 2026

**Date:** 29 August 2026
**Status:** Review only. Nothing in this document has been built, tested, or verified against a live run. Every claim about current flow behaviour is drawn from repository documents, not from Designer or Activity trace, and should be treated as unconfirmed until checked.
**Purpose:** David asked for a genuine outside view on the architecture — specifically, how this would be built if it were started today knowing everything now learned, including where a decision already made and documented looks wrong in hindsight.

**Sources read for this review:** `CURRENT-STATE.md`, `design-flow-c-chat-transcript-capture.md`, `design-idea-2026-08-28-onenote-lane-routing-via-category.md`, `handover-2026-08-28-recurring-chat-scoping.md`, `analysis-scope-2026-08-23-performance-and-architecture.md`, `analysis-scope-2026-08-23-naming-convention-audit.md`, `design-amendment-2026-08-20-per-occurrence-recurring-pages.md`, `phase-2-vision.md`, `strategic-review-agenda.md`, `flow-reference-2026-08-21-full-peek-code-capture.md`, and the `Crox2310/operating-model` README.

**Important caveat on staleness:** the most recent full Peek Code capture in the repo is dated 21 August and predates the 23 August sprint (BUG-01, FR-01, FR-02, FR-03, BUG-02, OutStatus differentiation). Flow A in particular has changed since. Sections 5 and 7 below lean on that capture and should be re-checked against current state.

---

## 1. Headline conclusions

1. The flow **count** is roughly right. The flow **boundaries** are wrong — they are drawn along the user journey, which does not compose when new writers are added.
2. Keep the system flow-heavy. Push only judgement tasks into model reasoning, and only where a wrong answer produces a wrong suggestion rather than a wrong write.
3. The OneNote lane-routing idea should not be built as automation. It solves a browsing problem; the stated retrieval need is a search problem, and the operating model already reached the opposite conclusion for email and files.
4. The SharePoint mapping list should become a cache rather than the source of truth. Once it is, retention length stops being risky.
5. Occurrence date is currently derived by parsing an HTML `<title>` out of a payload the system generated itself. This is a genuine design error rather than an accepted constraint, and it is now a prerequisite for Flow C.

---

## 2. Flow boundaries — split by capability, not by journey

The current split is Flow A (find the meeting) then Flow B (write the page). That is a journey boundary. It does not compose, because Flow C needs part of Flow B, pre-day prep needs the same part, and a future transcript writer needs it again. Each new writer either re-implements the logic or reaches into the largest and most corruption-prone flow in the system.

**Proposed shape — four to five flows, two of them shared children:**

| Flow | Owns | Contract |
|---|---|---|
| Resolve Meeting (today's Flow A) | Calendar search, candidate list, navigation state | in: date context, selection. out: candidates, occurrence identity |
| **Resolve Page** (extracted from Flow B) | The mapping list, section and page resolution. Nothing else touches SharePoint. | in: occurrence identity. out: pageId, webUrl, status |
| **Write Region** (extracted from Flow B) | Anchored insert into a named page region | in: pageId, region, HTML, mode. out: status |
| Capture Invite (Flow B, thinned) | Invite-specific extraction, then two child calls | — |
| Capture Chat (Flow C) | Chat extraction and summarisation, then two child calls | — |

**Why this and not something else:** the dominant cost over three months has not been latency or API limits. It has been the Designer value-field corruption pattern, twelve-plus documented incidents, blanking twenty-plus actions at a time. Blast radius scales with flow size. Splitting Flow B is the only structural change available that directly reduces it, and it is the same change that makes Flow C, pre-day prep, and transcript capture cheap instead of each being a fresh build.

**Honest cost:** each child flow adds a round trip, and there is currently no per-action timing data to say what that costs. Child flows also require solution-aware setup, which is on the backlog and unstarted. This is a trade of unmeasured latency for measured maintainability. Given twelve documented corruption incidents against zero measured latency complaints, the trade looks right — but it is a trade.

---

## 3. Agents versus flows

**Recommendation: stay flow-heavy.** Push exactly three things into model reasoning, each as a bounded call rather than an agent with tool access.

The operating principle: **the model proposes, the flow disposes.** Anything where a wrong answer produces a wrong *write* stays deterministic, because OneNote writes are hard to notice and harder to reverse. Anything where a wrong answer produces a wrong *suggestion* that the user then confirms can be model-driven.

- **Judge and summarise chat** — as already designed in `design-flow-c-chat-transcript-capture.md`. Correct.
- **Parse messy HTML** — as already designed, and worth extending to the invite body. The SD12 fix in `handover-2026-08-28-recurring-chat-scoping.md` (search for the URL pattern rather than an anchor id) is a second-generation patch on the same class of problem the Flow C design already decided to hand to an LLM. A third invite template will break it again. Keep the `indexOf` version as a deterministic fallback, since it is on the critical path.
- **Ambiguous meeting reference** — fetch the day's candidates deterministically, let the model rank them against the phrasing used, present its pick for confirmation, then pass a hard identifier onward.

Everything else stays in flow logic. The P/N/date navigation state machine emphatically stays there: it took weeks to stabilise, and state machines are what LLMs are worst at holding.

**The tradeoff, stated plainly:** agent reasoning buys flexibility at the cost of reproducibility. This project's entire working method — Peek Code, Activity traces, known-good-values reference files, scratch-flow verification — is built on reproducibility. Moving logic into agent reasoning deletes the artefact that method operates on. That is an unusually strong argument for staying flow-heavy *in this project specifically*. It would not apply to a project without this debugging discipline.

---

## 4. Findability — disagreement with the lane-routing design

**Recommendation: do not build the lane-routing automation described in `design-idea-2026-08-28-onenote-lane-routing-via-category.md`.** The manual reorg into section groups, which that document correctly splits off as an independent track, is worth doing for visual relief. The automation is not.

**Reason 1 — the stated query defeats it.** The retrieval need David described is "that thing Rich Beaumont mentioned about CCT in some meeting a while back." That is person plus topic plus fuzzy time. A section-group hierarchy answers none of the three. It answers "which lane", which is the one fact you would have to already know, and the one most likely to be ambiguous: Rich sits in Priority, a CCT discussion could sit in Supply Chain Tech or Cross Domain, and the meeting could plausibly be filed under any of the three.

**Reason 2 — lane is a time-varying label being encoded as permanent structure.** The operating-model README states that the burst-derived half of Priority has membership shifting week to week. A meeting captured during a September burst would live in the Priority section group permanently, including after the burst ends and the content is obviously Supply Chain Tech. Correcting that means moving sections, which is exactly the re-sorting cost the operating model exists to avoid.

**Reason 3 — the operating model already reached the opposite conclusion.** Part A Stage 2 says do not add folders beyond the fixed set, prefer search within a lane over sub-organisation, and leave old folders in place rather than migrating. Part C Stage 2 says trust search over browsing. Lane-first hierarchy in OneNote contradicts the conclusion the same document reached for email and files, for the same underlying reason.

**What to do instead**, in descending order of leverage per unit of effort:

1. **Put the retrieval keys in the page text.** OneNote's index reaches page content, not folder position. The query resolves only if "Rich Beaumont" and "CCT" both appear as text on a page. The attendee list is available from the calendar event and does not currently appear to be written to the page. Adding attendees, lane, series name and date as page text is worth more for retrieval than the entire section-group build.
2. **Tag the lane as text, not as structure.** Searchable, combinable with other terms, and correctable with a keystroke when a burst ends.
3. **Add one rolling index page.** One appended line per capture: date, series, lane, attendees, link. Gives browsing and search at once, costs one append rather than a rewrite of every section creation and lookup action, and degrades gracefully when the lane was wrong.

---

## 5. State and coordination

**Keep SharePoint.** It is the right choice. Dataverse buys typing that is not needed at a licensing cost that is not wanted; a JSON blob has no concurrency story; a Topic variable does not persist.

**Make the list a cache, not the source of truth.** Today, when `Filter_Existing_Mapping` finds nothing, Flow B routes to `CREATE_REQUIRED` and makes a new page. A missing row therefore produces a silent duplicate page rather than a failure. `Filter_Pages_By_Title` already exists in Flow B (built for FB-04a, noted as never exercised by a live run). Wiring it into the create path so a missing row falls back to a OneNote title lookup makes the whole class of failure survivable, and makes retention safe at any length.

**Fix the `<title>` scrape.** Flow B deriving the occurrence date by parsing an HTML `<title>` out of a payload Flow A generated is the one item in the build best described as a design error. `design-amendment-2026-08-20-per-occurrence-recurring-pages.md` flagged the alternative — pass the date as its own field — and it was not done. Flow C needs the occurrence date to look up the mapping row, so this is now a prerequisite, not an improvement.

**Replace the positional trigger contract.** `text`, `text_1` through `text_5`, where `text` is `IsRecurring` and `text_1` is `MeetingTitle`, is what invited the field-swap slip caught on 23 August. See `addendum-2026-08-29-contract-naming.md`.

**Lesson from BUG-01's root cause.** The unique constraint on `SeriesMasterId` alone was the schema asserting "one page per series" after the design had moved to per-occurrence. Enforce the composite (`SeriesMasterId` + `OccurrenceDate`) or enforce nothing. A uniqueness constraint on one column of a composite key is a latent bug by construction.

**The DLP detour left an unexploited asset.** "Send a Microsoft Graph HTTP request" via the Teams connector is a DLP-approved Graph escape hatch. If it will pass OneNote endpoints, `PATCH /onenote/pages/{id}/content` with `target` and `action: replace` becomes available. Emitting `data-id` attributes on the four section headers at page creation would then make Flow C's "replace my own prior chat summary" a single targeted PATCH rather than string surgery on retrieved HTML — removing the most fragile item in the Flow C build. **Unconfirmed. Test in scratch before designing around it.**

---

## 6. Critique of the Flow C design

Most of `design-flow-c-chat-transcript-capture.md` holds up. The separate-flow rationale, the four-section page layout, the explicit "nothing to add" marker rather than a silent skip, and feeding raw HTML to the model rather than hand-stripping it are all the right calls. Three changes:

**The pagination gap should close before build, not after.** The document's own framing is exact: the current behaviour produces the right outcome for the wrong reason, because only page one is ever fetched. The fix is small — loop on `@odata.nextLink`, exit early on the first message predating the occurrence start. Building on a known-unsound foundation produces an unreproducible bug on a chatty recurring meeting some weeks later.

**Make the AI step return structured JSON with an explicit boolean.** One call is right, but a single prompt doing two jobs drifts toward summarising everything, because producing content is what the model is pulled toward. Require `{noteworthy: bool, reason: string, notes: {...}}` and branch deterministically on the boolean. Same cost, and it produces the audit trail this project's method depends on.

**The on-demand trigger is the right first build and the wrong end state.** Part D of the operating model says the capture agent is what turns delegation into a free move. It is not free if it requires remembering a post-meeting action, and the chat summary is most valuable for meetings that were *not* attended — the ones least likely to be thought about afterwards. Target state is a scheduled evening sweep over mapping rows with today's `OccurrenceDate` and no chat-captured flag, with the Topic retained as the manual and retry path. Build on-demand first, but put the work in a child flow taking an occurrence identity so the trigger is swappable.

---

## 7. Performance — structural findings from the 21 August capture

Ranked by expected value. All derived from `flow-reference-2026-08-21-full-peek-code-capture.md` and therefore possibly stale.

**1. Flow A runs twice per capture, and the second run is nearly all waste.** The Topic calls `C2_Call_FlowA_Initial` to build the list, then calls the same flow again via `invokeFlowAction_bIIKPf` after the user picks a number. That second call re-fetches the whole day's calendar from Graph and re-runs the filter and sort purely to pick one item out of a list returned seconds earlier. Every P or N press is another full round trip on top. Two navigation presses before selecting means four Graph calendar calls to capture one meeting.

**2. The five-second delay may not need to be on the critical path.** `Compose_PageSelfUrl_Created` reads the self URL straight from `Create_OneNote_Page`'s own output, before the delay. If the user-facing link can be derived from that, the Response action can move up to immediately after page creation, with the delay, title fix and mapping write completing behind it. A Request/Response flow keeps executing after Response fires.

**3. The title fix may be removable entirely, taking the delay with it.** The Topic already builds `<title>{Topic.PageTitle}</title>` into `text_3`, and in OneNote's API the `<title>` element of posted HTML is the page title. The existence of a post-creation `Set_PageTitle_Recurring` suggests the connector's Create action overrides or ignores it. One scratch test settles it. If it holds, `Set_PageTitle_Recurring`, the delay, `Get_Pages_In_Section_PostCreate` and its filter all leave the create path.

**4. `Get_items` is unfiltered — a correctness problem before a performance one.** No `$filter`, no `$top`, filtered client-side. Since BUG-01 moved to one row per occurrence, the list grows roughly one row per captured meeting. The SharePoint connector's default page size is 100 unless pagination is enabled. Past a hundred rows, `Filter_Existing_Mapping` begins missing rows that exist, and the symptom is a duplicate page rather than an error.

**5. Eight `InitializeVariable` actions chained sequentially at the top of Flow B.** Power Automate charges roughly 50–150ms of orchestration per action regardless of what it does. Individually trivial, collectively not, and Flow B has dozens of Composes beyond those.

**What Peek Code cannot answer.** Timing lives in Activity traces. But the Topic-side hops between three `InvokeFlowAction` calls appear in neither, and in an architecture this shape they are a strong candidate for the largest single chunk. Measurement without new instrumentation: compare the end timestamp of Flow A run two against the start timestamp of Flow B in run history. That gap is pure Topic overhead with no human in the loop.

---

## 8. Future capabilities worth building

**A "search my meeting notes" Topic.** Reads the enriched index, uses the model to rank and answer, returns links to pages. This is the direct answer to the Rich-Beaumont-CCT query and needs no new infrastructure once the index carries summary text. Tradeoff: only as good as what is indexed, and the index stops being small, so query filtering becomes load-bearing rather than an optimisation.

**Pre-day prep (Phase 2 Feature 2).** Highest return of anything in `phase-2-vision.md`, and cheap once Resolve Page is a child flow, because pre-day prep is a loop around a call that already exists. Removes the "did I capture that" question entirely. Tradeoff: pages for meetings that never generate notes, so reuse the FR-02 exclusion filter and consider gating on lane category or attendee count.

**A rolling actions page.** The chat summariser already extracts actions. Leaving them on forty separate pages means they are captured but not actionable. One appended line per meeting is nearly free. **Real risk worth naming:** this competes with the task-creation habit in the operating model's Part A batch block, and two places where actions live is worse than one. Only build it if one of the two is made authoritative.

---

## 9. What in this document is unverified

- Everything in Section 7 derives from a 21 August capture that predates the 23 August sprint.
- The claim that the Teams Graph HTTP action may reach OneNote endpoints is untested.
- The claim that `<title>` sets the page title at creation is untested.
- The claim that the Topic collapses six `OutStatus` values into two rests on the 21 August Topic YAML, where `C11_Check_OutStatus` branches on `OutStatus = "OK"` against a single generic error message. If the Topic was updated alongside Flow B on 23 August, this is already resolved.
- Whether attendees currently appear in page content has not been checked.

---
*Written 29 August 2026 as a review session, not a build session. No flow, Topic, or list was modified in producing it.*
