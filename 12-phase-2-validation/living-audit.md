# Living Audit — Per-Action Expression Catalogue

See `PROCESS-expression-audit-maintenance.md` for the maintenance rules governing this document. Short version: this is current ground truth, not a session log. Update it the moment an expression changes in Designer, before closing out the session's handover note.

**Last updated:** 2026-06-25
**Coverage:** Flow B (`PA - Resolve OneNote Meeting Section - v2 Clean Build`, flowId `ed112c88-b94b-f111-bec6-002248a38052`) — primary path complete. Flow A (`PA - Resolve Meeting Selection - v1 Clean Build`, flowId `d9d7ccf7-7d61-f111-a826-6045bde03856`) — fully confirmed, no remaining gaps.

**Status key:** 🔴 confirmed bug, not fixed · 🟡 suspect/unconfirmed · 🟢 confirmed fixed and tested · ⚪ confirmed clean (no issue)

**Bug pattern reference**: (1) string-wrapped booleans / missing `empty()` guards, (2) blank or literal-`''` values, (3) wrong/mismatched field names, (4) missing `@` expression prefix, (5) **wrong-array indexing — FA19: indexing an unfiltered array using an index/count derived from a filtered array.**

---

## Flow B — Trigger

### `When an agent calls the flow` ⚪ confirmed clean
Mapping confirmed via Topic YAML (`living-audit-topic.md` Section 5): `text`=IsRecurring, `text_1`=MeetingTitle, `text_2`=SeriesMaster, `text_3`=PageHtml, `text_4`=MeetingId.

---

## Flow B — Condition_IsRecurring

### `Condition_IsRecurring` 🟢 confirmed fixed and published 2026-06-25
```
toLower(string(triggerBody()?['text'])) is equal to 'true'
```
Fix applied via Code view — expression JSON corrected to:
```json
"equals": [
  "@toLower(string(triggerBody()?['text']))",
  "'true'"
]
```
Previously the entire condition sentence was stuffed as a string literal into the first `equals` element, causing a publish-time InvalidExpression error. Fixed by editing Code view JSON directly.

### True branch
**`Compose Input SeriesMasterId`** ⚪ **`Compose Input MeetingTitle`** ⚪ **`Filter Existing Mapping`** ⚪ — SeriesMasterId equal to SeriesMasterId (dynamic). **`Compose ExistingPageSelfUrl`** ⚪ **`Compose PageDecision`** ⚪ **`Compose Match Count`** ⚪
**`varFinalExistingPageSelfUrl_1`** ⚪ — `@outputs('Compose_ExistingPageSelfUrl')` confirmed in Code view.
**`varFinalPageDecision_1`** ⚪ — `@outputs('Compose_PageDecision')` confirmed in Code view.
**`varFinalMatchCount_1`** ⚪ — `@outputs('Compose_Match_Count')` confirmed in Code view.

### False branch (one-off path)
**`FB-F01`** ⚪ **`Get Sections OneOff`** ⚪ **`Filter OneNote Section OneOff`** ⚪ **`Compose Section Match Count OneOff`** ⚪
**`Condition Section Exists OneOff`** 🟡 — `greater(...) equal to @true` pattern, same family as other conditions. Not yet re-expanded this session to confirm fix status.
True: **`For each 1`** ⚪
- **`Set varTargetSectionPagesUrl OneOff Exists`** 🟢 fixed 2026-06-25 — `items('For_each_1')?['pagesUrl']`
- **`Set varOneNoteResolverResult Exists OneOff`** 🟢 fixed 2026-06-25 — `ExistingSection`
False: **`Create Section OneOff`** ⚪
- **`Set varTargetSectionPagesUrl OneOff Created`** 🟢 fixed 2026-06-25 — `outputs('Create_Section_OneOff')?['body']?['pagesUrl']`
- **`Set varOneNoteResolverResult Created OneOff`** 🟢 fixed 2026-06-25 — `CreatedSection`

---

## Flow B — Condition Mapping Exists

### `Condition Mapping Exists` ⚪ working pattern
```
if(empty(coalesce(variables('varFinalMatchCount'), '')), '0', greater(int(coalesce(variables('varFinalMatchCount'), '0')), 0)) is equal to true
```

True: **`Compose Branch Result`** ⚪, **`Set varTargetSectionPagesUrl ExistingMapping`** ⚪, **`Set varOneNoteResolverResult ExistingMapping`** ⚪.

False:
**`Compose Branch Result NoMatch`** ⚪
**`Condition_Should_Write_Mapping`** 🟢 confirmed fixed and published 2026-06-25
```
greater(int(if(empty(variables('varFinalMatchCount')), '0', variables('varFinalMatchCount'))), 0)
```
Previously crashed with `int('')` when `varFinalMatchCount` was empty string. Fixed with `empty()` guard.
True branch: **`Send_an_HTTP_request_to_SharePoint`** ⚪ clean. POST `_api/web/lists/GetByTitle('RecurringMeetingSectionMap')/items`.
False: empty, by design.
**`Compose IgnoreSeriesMasterId`** 🟡 — literal `''` pattern, low priority, not yet fixed.
**`Compose PageRoute CreateRequired`** ⚪ **`Compose SectionDisplayName`** ⚪ **`Compose SafeSectionName`** ⚪

---

## Flow B — True-branch page-creation chain

### `Condition Should Create Page` 🟡 unconfirmed, suspect — same family as other conditions, not yet expanded.

True: **`Create OneNote Page`** ⚪, **`Compose PageSelfUrl Created`** ⚪, **`HTTP Update SP PageSelfUrl`** ⚪
- **`Set varPageAction Created`** 🟡 — not confirmed blank or fixed this session, needs check
- **`Set varOutputPageSelfUrl Created`** 🟢 fixed 2026-06-25 — `outputs('Compose_PageSelfUrl_Created')`
- **`Compose UpdateHtmlFragment`** ⚪, **`Compose ExistingPageId`** ⚪
- **`Set varOutputPageLink Created`** 🟢 confirmed already fixed — `outputs('Create_OneNote_Page')?['body']?['links']?['oneNoteWebUrl']?['href']`

### Condition Is Genuine Existing Page ⚪ clean, both branches present.
True: **`Get Sections Existing Branch`** ⚪, **`Create Page OneOff`** ⚪, **`Set varOutputPageLink Created OneOff`** 🟡 — not confirmed this session.
False: **`Filter Existing Section By Name`** ⚪, **`Apply to each Existing Section`** ⚪ → **`Update page content Existing Branch`** ⚪
- **`Set varPageAction UpdatedAppend`** 🟡 — not confirmed this session
- **`Set varOutputPageLink Existing`** 🟢 fixed 2026-06-25 — `outputs('Compose_ExistingPageSelfUrl')`

---

## Flow B — Final response

**`Compose AgentResponseSummary`** ⚪ logic clean.
**`Compose SP Item Count`** ⚪
**`Respond to the agent`** ⚪ — full 20-output schema, see `living-audit-topic.md` Section 5 for the complete confirmed list.

---

## Flow B-wide issues

🟡 **`runAfter` casing** — `"SUCCEEDED"` vs `"Succeeded"`. **STATUS UNRESOLVED:** Flow A and Flow B actions use `"SUCCEEDED"` consistently in Code view. Do not action any fix until casing is definitively confirmed against Power Automate's actual requirement. Not causing live failures currently.
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
**`FA03`** 🟢 confirmed fixed — verified via Designer: dynamic content chip showing `triggerBody()?['OriginalUserSearchText']` correctly bound. Was previously listed as 🔴 missing `@` prefix but fix was already applied before this session.
**`FA04`** 🟢 confirmed fixed — verified via Designer: `DateContext` chip correctly bound to `triggerBody()?['DateContext']`. Same as FA03.
**`FA05`** ⚪ — correctly prefixed.
**`FA03A DEBUG`** ⚪
**`FA06`** ⚪ **`FA07`** ⚪ — StartOfDayUtc/EndOfDayUtc, clean.
**`FA08`** ⚪ — date range correct; `calendarId` hardcoded (design note, not a bug).
**`FA08A DEBUG`** ⚪ **`FA09`** ⚪ — `body(...)?['value']`, clean.

---

## Flow A — Candidate filtering and array build (FA09A–FA13)

**`FA09A Filter CandidatesByTitle`** 🟡 suspect, likely dormant — fallback to `text_2` (OriginalUserSearchText) if `text` (UserSearchText) empty; fallback is dead code under normal conditions.
**`FA10`** ⚪ — empty array init.
**`FA12 Append to array varCandidates`** 🟡 — `IsRecurring` derived from `equals(item()?['type'], 'occurrence')` only; misses `exception` and `seriesMaster` types. Needs confirming whether FA28A's seriesMasterId-presence check supersedes this field downstream.
**`FA13`** ⚪ — `length(body('FA09A...'))`, clean.

---

## Flow A — Match-count branching (FA27–FA43B)

**`FA27`** ⚪ `equals(outputs('FA13...'), 0)`. NO_MATCH branch (FA27B–H) — all ⚪ clean.
**`FA27A`** ⚪ `equals(outputs('FA13...'), 1)`.

Single-match (true): **`FA28`** ⚪, **`FA28A`** 🟢 confirmed fixed live, **`FA28B`** 🟢 confirmed fixed live, **`FA29`** ⚪, **`FA30`** ⚪, **`FA31`** ⚪
**`FA32`** 🟢 confirmed fixed and published 2026-06-25 — `string('')` expression chip confirmed in Designer.

Multi-match (false): **`FA33A`** 🟢 fixed live, **`FA34A`** 🟢 fixed live, **`FA35`/`FA36`/`FA37`** ⚪ all clean, **`FA38`/`FA39`/`FA40`** ⚪ clean, **`FA41`/`FA42`/`FA43A`/`FA43B`** ⚪ all clean.

---

## Flow A — FA14–FA26

**`FA14 Compose CandidateList`** ⚪ **`FA15 Compose IsSelectionMode`** ⚪ **`FA17 Condition IsSelectionMode`** ⚪ **`FA16 Compose SelectedIndex`** ⚪ **`FA18 Condition SelectedIndexInRange`** ⚪

**`FA19 Compose SelectedEvent`** 🟢 confirmed fixed and published 2026-06-25
```
outputs('FA09A_Filter_CandidatesByTitle')[outputs('FA16_Compose_SelectedIndex')]
```
Previously indexed `FA09_Compose_CandidateArray` (unfiltered) — now correctly indexes `FA09A_Filter_CandidatesByTitle` (filtered). Verified via Designer expression editor tooltip.

**`FA20 Compose OutMeetingTitle`** ⚪ **`FA21 Compose OutCalendarEventId`** ⚪ — clean; now correctly inherit from fixed FA19.

**`FA22 Compose OutMatchCount_Resolved`** ⚪ — literal `1`, clean.

**`FA23 Compose OutCandidateList_Resolved`** 🟢 confirmed fixed and published 2026-06-25 — `string('')` expression chip confirmed in Designer.

Error/out-of-range branch: **`FA24`** ⚪ **`FA25`** ⚪ **`FA26`** ⚪ — all clean.

**`FA19B Compose OutIsRecurring Resolved`** 🟢 confirmed fixed and published 2026-06-25
```
if(empty(coalesce(outputs('FA19_Compose_SelectedEvent')?['seriesMasterId'], '')), 'false', 'true')
```
Verified via Designer expression editor tooltip.

**`FA19C Compose OutSeriesMasterId Resolved`** 🟢 confirmed fixed and published 2026-06-25
```
coalesce(outputs('FA19_Compose_SelectedEvent')?['seriesMasterId'], '')
```
Verified via Designer expression editor tooltip.

---

## Flow A — FA43 Respond to agent

**`FA43 Respond to agent`** ⚪ fully confirmed clean 2026-06-25.

Seven-field output schema: status, matchcount, candidatelist, meetingtitle, calendareventid, isrecurring, seriesmasterid.

**`IsRecurring`** — `outputs('FA19B_Compose_OutIsRecurring_Resolved')` confirmed via Designer tooltip. Direct reference to FA19B (no coalesce needed — FA19B always returns 'false' or 'true').

**`SeriesMaster`** — `outputs('FA19C_Compose_OutSeriesMasterId_Resolved')` confirmed via Designer tooltip. Direct reference to FA19C.

All other fields coalesce across appropriate branch outputs — previously audited as clean.

---

## Open items / not yet covered by this audit

- `FA12`'s IsRecurring derivation — confirm whether FA28A's seriesMasterId-presence check supersedes this field downstream before prioritising a fix.
- `FA09A`'s fallback-source risk — tied to the Topic-level `OriginalUserSearchText` rebinding issue in `living-audit-topic.md`.
- `Condition Section Exists OneOff` — `greater(...) equal to @true` pattern not yet re-expanded; mark as 🟡 pending verification.
- `Set varPageAction Created`, `Set varPageAction UpdatedAppend`, `Set varOutputPageLink Created OneOff` — not confirmed clean or broken this session; needs a check pass.
- `Compose IgnoreSeriesMasterId` — literal `''`, low priority, not yet fixed.
- `runAfter` casing — unresolved, see Flow B-wide issues.
- Live UJ1 re-test pending — both flows published 2026-06-25, connections confirmed Connected. Test in a brand new Teams thread.
- UJ2, UJ3, UJ4 — all Flow A fixes now applied; UJ2/UJ3/UJ4 are theoretically unblocked pending UJ1 baseline confirmation.
