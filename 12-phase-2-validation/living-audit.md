# Living Audit — Per-Action Expression Catalogue

See `PROCESS-expression-audit-maintenance.md` for the maintenance rules governing this document. Short version: this is current ground truth, not a session log. Update it the moment an expression changes in Designer, before closing out the session's handover note.

**Last updated:** 2026-06-21
**Coverage:** Flow B (`PA - Resolve OneNote Meeting Section - v2 Clean Build`, flowId `ed112c88-b94b-f111-bec6-002248a38052`) — primary path complete. Flow A (`PA - Resolve Meeting Selection - v1 Clean Build`, flowId `d9d7ccf7-7d61-f111-a826-6045bde03856`) — trigger through FA26, plus FA27 through FA43B, fully confirmed. No remaining gap in Flow A's primary path.

**Status key:** 🔴 confirmed bug, not fixed · 🟡 suspect/unconfirmed · 🟢 confirmed fixed and tested · ⚪ confirmed clean (no issue)

**Bug pattern reference**: (1) string-wrapped booleans / missing `empty()` guards, (2) blank or literal-`''` values, (3) wrong/mismatched field names, (4) missing `@` expression prefix, (5) **wrong-array indexing — added 2026-06-21, see Flow A FA19 below: indexing an unfiltered array using an index/count derived from a filtered array.**

---

## Flow B — Trigger

### `When an agent calls the flow` 🟢 resolved
Mapping confirmed via Topic YAML (`living-audit-topic.md` Section 5): `text`=IsRecurring, `text_1`=MeetingTitle, `text_2`=SeriesMaster, `text_3`=PageHtml, `text_4`=MeetingId.
**Fix (confirmed correct, not yet applied):** `Condition_IsRecurring` should read `triggerBody()?['text']`, not `['IsRecurring']`.

---

## Flow B — Condition_IsRecurring

### `Condition_IsRecurring` 🔴
```
toLower(string(triggerBody()?['IsRecurring'])) is equal to "true"
```
Fix: read `triggerBody()?['text']` instead.

### True branch
**`Compose Input SeriesMasterId`** ⚪ **`Compose Input MeetingTitle`** ⚪ **`Filter Existing Mapping`** ⚪ — SeriesMasterId equal to SeriesMasterId (dynamic). **`Compose ExistingPageSelfUrl`** ⚪ **`Compose PageDecision`** ⚪ **`Compose Match Count`** ⚪
**`varFinalExistingPageSelfUrl_1`** 🔴 blank. **`varFinalPageDecision_1`** 🔴 blank. **`varFinalMatchCount_1`** 🔴 blank (`""`) — root cause feeding `Condition_Should_Write_Mapping` crash.

### False branch
**`FB-F01`** ⚪ **`Get Sections OneOff`** ⚪ **`Filter OneNote Section OneOff`** ⚪ **`Compose Section Match Count OneOff`** ⚪
**`Condition Section Exists OneOff`** 🔴 — `greater(...)` equal to true, needs guard check same as `Condition_Should_Write_Mapping`.
True: **`For each 1`** ⚪, **`Set varTargetSectionPagesUrl OneOff Exists`** 🔴 blank, **`Set varOneNoteResolverResult Exists OneOff`** 🔴 blank.
False: **`Create Section OneOff`** ⚪, **`Set varTargetSectionPagesUrl OneOff Created`** 🔴 blank, **`Set varOneNoteResolverResult Created OneOff`** 🔴 blank.

---

## Flow B — Condition Mapping Exists

### `Condition Mapping Exists` ⚪ working pattern
```
if(empty(coalesce(variables('varFinalMatchCount'), '')), '0', greater(int(coalesce(variables('varFinalMatchCount'), '0')), 0)) is equal to true
```

True: **`Compose Branch Result`** ⚪, **`Set varTargetSectionPagesUrl ExistingMapping`** ⚪, **`Set varOneNoteResolverResult ExistingMapping`** ⚪.

False:
**`Compose Branch Result NoMatch`** ⚪
**`Condition_Should_Write_Mapping`** 🔴 **CONFIRMED ROOT CAUSE OF LIVE CRASH (2026-06-20)**
```json
{"type":"If","expression":{"and":[{"equals":["@greater(int(coalesce(variables('varFinalMatchCount'),'0')),0)","@true"]}]}}
```
**Bug:** `coalesce(var, '0')` only substitutes on null, not empty string. `varFinalMatchCount_1` is `""`. `int('')` throws.
**Fix:** `greater(int(if(empty(variables('varFinalMatchCount')), '0', variables('varFinalMatchCount'))), 0)`
True branch: **`Send_an_HTTP_request_to_SharePoint`** ⚪ clean once guard fixed. POST `_api/web/lists/GetByTitle('RecurringMeetingSectionMap')/items`, body `{Title, SeriesMasterId, MeetingTitle, SectionPagesUrl, Status}`. runAfter casing — see Flow B-wide issues, status now unresolved.
False: empty, by design.
**`Compose IgnoreSeriesMasterId`** 🔴 — literal `''`.
**`Compose PageRoute CreateRequired`** ⚪ **`Compose SectionDisplayName`** ⚪ **`Compose SafeSectionName`** ⚪

---

## Flow B — True-branch page-creation chain

### `Condition Should Create Page` 🟡 unconfirmed, suspect — same family as other conditions, not yet expanded.

True: **`Create OneNote Page`** ⚪, **`Compose PageSelfUrl Created`** ⚪, **`HTTP Update SP PageSelfUrl`** ⚪, **`Set varPageAction Created`** 🔴 blank, **`Set varOutputPageSelfUrl Created`** 🔴 blank, **`Compose UpdateHtmlFragment`** ⚪, **`Compose ExistingPageId`** ⚪, **`Set varOutputPageLink Created`** 🔴 blank (regressed from this morning's fix — re-apply `outputs('Create_Page_OneOff')?['body']?['links']?['oneNoteWebUrl']?['href']`).

### Condition Is Genuine Existing Page ⚪ clean, both branches present.
True: **`Get Sections Existing Branch`** ⚪, **`Create Page OneOff`** ⚪, **`Set varOutputPageLink Created OneOff`** 🔴 blank — same intended fix as above.
False: **`Filter Existing Section By Name`** ⚪, **`Apply to each Existing Section`** ⚪ → **`Update page content Existing Branch`** ⚪ confirmed clean. **`Set varPageAction UpdatedAppend`** 🔴 blank, **`Set varOutputPageLink Existing`** 🔴 blank.

---

## Flow B — Final response

**`Compose AgentResponseSummary`** ⚪ logic clean, but always falls through to generic message since `varPageAction` is never set upstream — self-resolves once those fixes land.
**`Compose SP Item Count`** ⚪
**`Respond to the agent`** ⚪ — full 19-output schema, see `living-audit-topic.md` Section 5 for the complete confirmed list.

---

## Flow B-wide issues

🔴 **`runAfter` casing** — `"SUCCEEDED"` vs `"Succeeded"`. **STATUS UNRESOLVED (2026-06-21):** Flow A's actions also consistently use `"SUCCEEDED"`. Do not action any fix until casing is definitively confirmed against Power Automate's actual requirement.
🟡 **Naming/type concern** — `sectionId` fields receive a variable named `varTargetSectionPagesUrl`; not confirmed broken.
⚪ **Dead conditional logic** — `Set_varTargetSectionPagesUrl_ExistingMapping`'s branches are identical, low priority.

---

## Flow A — Trigger

### `When an agent calls the flow` ⚪ confirmed clean
Self-documented schema: `text`=UserSearchText, `text_1`=InSelectedNumber, `text_2`=OriginalUserSearchText, `text_3`=DateContext, `text_4`=MaxCandidates. Matches Topic bindings exactly. Topic sends `InSelectedNumber` as literal `" "` on first call to satisfy the `required` constraint — deliberate, not a bug.

---

## Flow A — Initial variable setup (FA01–FA10)

**`FA01`** ⚪ — `triggerBody()?['UserSearchText']`.
**`FA02`** ⚪ — defensively guarded against missing/empty/quoted-empty, good reference pattern.
**`FA03`** 🔴 **missing `@` prefix** — `"value": "triggerBody()?['OriginalUserSearchText']"` is a literal string, not evaluated. Fix: add `@`.
**`FA04`** 🔴 **missing `@` prefix, same bug** — `"value": "triggerBody()?['DateContext']"`. Fix: add `@`.
**`FA05`** ⚪ — correctly prefixed; confirms FA03/FA04 are isolated, not a sweep.
**`FA03A DEBUG`** ⚪ — would have surfaced FA03/FA04 in run history.
**`FA06`** ⚪ **`FA07`** ⚪ — StartOfDayUtc/EndOfDayUtc, clean.
**`FA08`** ⚪ — date range correct; `calendarId` hardcoded (design note, not a bug, but a fragility if ever generalized).
**`FA08A DEBUG`** ⚪ **`FA09`** ⚪ — `body(...)?['value']`, clean.

---

## Flow A — Candidate filtering and array build (FA09A–FA13)

**`FA09A Filter CandidatesByTitle`** 🔴 suspect, likely dormant — fallback to `text_2` (OriginalUserSearchText) if `text` (UserSearchText) empty; UserSearchText always populated currently so fallback is dead code under normal conditions, but interacts with the Topic-level `OriginalUserSearchText` rebinding risk noted in `living-audit-topic.md`.
**`FA10`** ⚪ — empty array init.
**`FA12 Append to array varCandidates`** 🔴 — `IsRecurring` derived from `equals(item()?['type'], 'occurrence')` only; misses `exception` and `seriesMaster` types. This is the historically "parked" FA12 fix, now root-caused. Needs confirming whether downstream consumes this field or whether FA28A's later seriesMasterId-presence check supersedes it.
**`FA13`** ⚪ — `length(body('FA09A...'))`, clean.

---

## Flow A — Match-count branching (FA27–FA43B)

**`FA27`** ⚪ `equals(outputs('FA13...'), 0)`. NO_MATCH branch (FA27B–H) — all ⚪ clean.
**`FA27A`** ⚪ `equals(outputs('FA13...'), 1)`.

Single-match (true): **`FA28`** ⚪, **`FA28A`** 🟢 confirmed fixed live, **`FA28B`** 🟢 confirmed fixed live, **`FA29`** ⚪, **`FA30`** ⚪, **`FA31`** ⚪, **`FA32`** 🔴 literal `''` (fix: `@string('')`).

Multi-match (false): **`FA33A`** 🟢 fixed live, **`FA34A`** 🟢 fixed live, **`FA35`/`FA36`/`FA37`** ⚪ all clean, **`FA38`/`FA39`/`FA40`** ⚪ clean, **`FA41`/`FA42`/`FA43A`/`FA43B`** ⚪ all clean (`@string('')`).

---

## Flow A — FA14–FA26 (the previously-open gap, now fully reviewed 2026-06-21)

**`FA14 Compose CandidateList`** ⚪ — `string(variables('varCandidates'))`, stringifies the full enriched array (distinct from FA40's human-readable `varCandidateListText`); purpose downstream not yet fully traced.

**`FA15 Compose IsSelectionMode`** ⚪ — `not(empty(trim(variables('varInSelectedNumber'))))`. Well-guarded, consistent with FA02's defensive init.

**`FA17 Condition IsSelectionMode`** ⚪ — `equals(outputs('FA15...'), true)`, clean boolean evaluation.

**`FA16 Compose SelectedIndex`** ⚪ (reviewed in detail, concluded not a live bug) — converts 1-based user input to 0-based index via `sub(int(...), 1)`; the empty-string-returns-0 branch is unreachable dead code given FA17/FA15's gating, so not itself defective.

**`FA18 Condition SelectedIndexInRange`** ⚪ — range-checks `FA16`'s index against `0` and `FA13`'s MatchCount; the `int()` wrapping around an already-numeric value is redundant but harmless.

**`FA19 Compose SelectedEvent`** 🔴 **HIGH-SEVERITY CONFIRMED BUG, NEW PATTERN (wrong-array indexing)**
```
outputs('FA09_Compose_CandidateArray')[outputs('FA16_Compose_SelectedIndex')]
```
Indexes into `FA09_Compose_CandidateArray` — the **unfiltered** array straight from the calendar API — using an index derived from the user's selection against the **filtered** list (`FA09A_Filter_CandidatesByTitle`, whose length is `FA13`'s MatchCount, the number actually shown to the user as the candidate list). When the calendar returns more events than match the title filter, these two arrays have different lengths and different per-index contents. A user selecting "2" from a filtered 3-item candidate list can have FA19 silently pull the wrong event from the larger unfiltered array — wrong title, wrong recurring status, wrong everything — with no error thrown since the index is often still in-bounds for the larger array.
**This is very plausibly the root cause of UJ2 (multi-match selection) never having been successfully live-tested.**
**Fix (not yet drafted/applied):** change the source array to `outputs('FA09A_Filter_CandidatesByTitle')` (or its equivalent dynamic reference) so indexing is consistent with what FA13/FA18's range-check and the user-facing candidate list are actually built from.

**`FA20 Compose OutMeetingTitle`** ⚪ **`FA21 Compose OutCalendarEventId`** ⚪ — both clean expressions, but both inherit FA19's wrong-source-event bug since they read from `FA19_Compose_SelectedEvent`. **Confirmed user-facing via `FA43_Respond_to_agent` (see below) — these are the first coalesce option for `meetingtitle`/`calendareventid`, so FA19's wrong event reaches the actual agent response, not just an internal variable.**

**`FA22 Compose OutMatchCount_Resolved`** ⚪ — literal `1`, clean.

**`FA23 Compose OutCandidateList_Resolved`** 🔴 — `"inputs": "''"`, same literal-2-character-string bug family as FA32 and Flow B's `Compose_IgnoreSeriesMasterId`. Fourth confirmed instance of this exact pattern. Fix: `@string('')`.

Error/out-of-range branch (FA17's else): **`FA24 Compose OutStatus_InvalidSelection`** ⚪ literal `"ERROR"`, **`FA25 Compose OutMatchCount_Error`** ⚪ literal `0`, **`FA26 Compose OutCandidateList_Error`** ⚪ literal `"Selected number is out of range."` — all clean.

**FA14–26 is now fully confirmed.** No remaining gap in Flow A's primary path.

---

## Flow A — FA43 Respond to agent (confirmed 2026-06-21, verified directly against live Designer source)

**`FA43 Respond to agent`** — two findings, both now fully confirmed via screenshots of the actual expanded coalesce expressions in Designer (not just inferred from JSON).

Seven-field output schema: status, matchcount, candidatelist, meetingtitle, calendareventid, isrecurring, seriesmasterid. Each field `coalesce()`s across the three branch outcomes (selection-resolved / single-match / no-match / multi-match, varies by field) plus a default.

**Finding 1 — confirms FA19 is user-facing, not just internal.** `meetingtitle` and `calendareventid` both list FA20/FA21 (which read from the broken `FA19_Compose_SelectedEvent`) as the first coalesce option. When a user is in selection mode and FA19 pulls the wrong event from the unfiltered array, that wrong title and wrong event ID are exactly what reaches the agent response and the user in Teams. This raises FA19's priority further — confirmed end-to-end user impact, not just an internal inconsistency.

**Finding 2 — CONFIRMED BUG (verified via Designer screenshot, exact expression text): `isrecurring`/`seriesmasterid` missing the selection-resolved branch.**
```
IsRecurring:
coalesce(outputs('FA28A_Compose_OutIsRecurring'), outputs('FA27G_Compose_OutIsRecurring_NoMatch'), outputs('FA43A_Compose_OutIsRecurring_Multi'), 'false')

SeriesMaster:
coalesce(outputs('FA28B_Compose_OutSeriesMasterId'), outputs('FA27H_Compose_OutSeriesMasterId_NoMatch'), outputs('FA43B_Compose_OutSeriesMasterId_Multi'), string(''))
```
Both fields coalesce across single-match (FA28A/FA28B), no-match (FA27G/FA27H), and multi-match (FA43A/FA43B) sources — but the selection-resolved branch (FA17 true → FA18 true → FA19-23) has **no equivalent `OutIsRecurring`/`OutSeriesMasterId` action at all**. Confirmed two ways: by cross-checking the full FA19-23 action list (only `OutMeetingTitle`/FA20, `OutCalendarEventId`/FA21, `OutMatchCount_Resolved`/FA22, `OutCandidateList_Resolved`/FA23 exist there) and now by directly reading the expanded coalesce expressions in Designer, which contain no `FA19`-derived reference at all.

**Impact:** when a user selects a meeting from a multi-match candidate list (the UJ3/UJ4 recurring-meeting scenario this entire feature exists to support), `isrecurring` and `seriesmasterid` both fall through to their defaults (`'false'` and `''`) on every single selection, since none of FA28A/FA27G/FA43A or FA28B/FA27H/FA43B were computed on the execution path that actually ran. The agent will always report a selected meeting as non-recurring with no series master, regardless of the truth — separate from and in addition to FA19's wrong-event bug.

**Fix (not yet drafted):** add two new Compose actions inside the FA18 true branch (alongside FA19-23), e.g. `FA19B_Compose_OutIsRecurring_Resolved` and `FA19C_Compose_OutSeriesMasterId_Resolved`, using the same seriesMasterId-presence pattern as FA28A/FA28B:
```
if(empty(coalesce(outputs('FA19_Compose_SelectedEvent')?['seriesMasterId'], '')), 'false', 'true')
coalesce(outputs('FA19_Compose_SelectedEvent')?['seriesMasterId'], '')
```
then add both as the first coalesce option in FA43's `isrecurring`/`seriesmasterid` fields. **This fix is downstream of and depends on FA19's fix being applied first**, since it reads from the same `FA19_Compose_SelectedEvent` action.

This is now the second highest-priority Flow A item after FA19 itself, and directly explains why UJ3/UJ4 testing (recurring meetings via the multi-match selection path) would fail even once Flow B connectivity and FA19 are both fixed.

---

## Open items / not yet covered by this audit

- `FA19`'s fix needs drafting and applying — highest-priority new item from this session, likely blocking UJ2 from ever having worked correctly.
- `FA43`'s missing isrecurring/seriesmasterid on the selection-resolved branch — CONFIRMED via Designer screenshot 2026-06-21 (exact expression text verified, no FA19-derived source present in either coalesce chip). Second-highest-priority item, blocks correct recurring-meeting detection specifically in the UJ3/UJ4-via-multi-match-selection path. Depends on FA19's fix landing first.
- Several Flow B `⚪ clean, not yet transcribed verbatim` entries should be filled in with exact expression text next time opened.
- `runAfter` casing — unresolved, see Flow B-wide issues.
- `FA12`'s IsRecurring derivation — confirm downstream consumption before prioritizing a fix.
- `FA09A`'s fallback-source risk, tied to the Topic-level `OriginalUserSearchText` rebinding issue in `living-audit-topic.md`.
- Run-history check on FA03/FA03A to establish how long the missing-`@`-prefix bug has been live.
- Live re-test of all fixes via a brand-new Teams thread — **blocked as of 2026-06-21**: both Flow B call nodes (C8B, C10) show "Flow not found or is turned off" in Designer. See `living-audit-topic.md` Section 5/8 — highest-priority item across both documents, since nothing here is testable until resolved.
