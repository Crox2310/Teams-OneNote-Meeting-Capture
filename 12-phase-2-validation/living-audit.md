# Living Audit — Per-Action Expression Catalogue

See `PROCESS-expression-audit-maintenance.md` for the maintenance rules governing this document. Short version: this is current ground truth, not a session log. Update it the moment an expression changes in Designer, before closing out the session's handover note.

**Last updated:** 2026-07-27
**Coverage:** Flow B (`PA - Resolve OneNote Meeting Section - v2 Clean Build`, flowId `ed112c88-b94b-f111-bec6-002248a38052`) — primary path complete. Flow A (`PA - Resolve Meeting Selection - v1 Clean Build`, flowId `d9d7ccf7-7d61-f111-a826-6045bde03856`) — fully confirmed, no remaining gaps.

**Status key:** 🔴 confirmed bug, not fixed · 🟡 suspect/unconfirmed · 🟢 confirmed fixed and tested · ⚪ confirmed clean (no issue)

**Bug pattern reference**: (1) string-wrapped booleans / missing `empty()` guards, (2) blank or literal-`''` values, (3) wrong/mismatched field names, (4) missing `@` expression prefix, (5) wrong-array indexing — FA19: indexing an unfiltered array using an index/count derived from a filtered array, (6) **missing `value` key entirely on a `SetVariable` action named as though it sets one — found 2026-07-27, distinct from pattern (2)'s blank/literal-empty-string case.**

**⚠️ 2026-07-27 correction notice:** This document's entries for `Set varOneNoteResolverResult ExistingMapping`, `Set varOneNoteResolverResult Exists OneOff`, and `Condition Is Genuine Existing Page` were found on 2026-07-27 to be **inaccurate** — recorded here as ⚪/🟢 despite two of the three actually having no `value` key at all (bug pattern 6), leaving the condition structurally unreachable in the live, published flow. Either the 2026-06-25 fixes recorded below were never actually saved/published, or were lost afterward — cause not established. All three are now corrected below and independently re-verified live via Peek Code and a full run trace; see `2026-07-27-condition-is-genuine-existing-page-defect.md` and AMEND-2026-07-27-002 in `amendment-log.md`. **Given this document has been shown to record fixes not actually present in the live flow, treat any 🟢/⚪ entry dated 2026-06-25 or earlier with some caution until independently re-verified, rather than as guaranteed current — this audit's own accuracy guarantee has a known gap.**

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
- **`Set varTargetSectionPagesUrl OneOff Exists`** ⚪ confirmed clean 2026-07-27 (real expression `@items('For_each_1')?['pagesUrl']`) — this document's 2026-06-25 entry for this action was accurate.
- **`Set varOneNoteResolverResult Exists OneOff`** 🟢 fixed 2026-07-27 — found to have **no `value` key at all** at the start of the 2026-07-27 session (contradicting this document's prior 🟢 "fixed 2026-06-25" claim — see correction notice above); the 2026-06-25 fix recorded here was evidently never actually saved/published. Re-fixed and confirmed present with `"value": "ExistingSection"` via Peek Code by end of the 2026-07-27 session. See AMEND-2026-07-27-002.
False: **`Create Section OneOff`** ⚪
- **`Set varTargetSectionPagesUrl OneOff Created`** 🟢 fixed 2026-06-25 — `outputs('Create_Section_OneOff')?['body']?['pagesUrl']`
- **`Set varOneNoteResolverResult Created OneOff`** 🟢 fixed 2026-06-25 — `CreatedSection`

---

## Flow B — Condition Mapping Exists

### `Condition Mapping Exists` ⚪ working pattern
```
if(empty(coalesce(variables('varFinalMatchCount'), '')), '0', greater(int(coalesce(variables('varFinalMatchCount'), '0')), 0)) is equal to true
```

True: **`Compose Branch Result`** ⚪, **`Set varTargetSectionPagesUrl ExistingMapping`** ⚪ confirmed clean 2026-07-27 (real expression pulling `SectionPagesUrl` from filtered mapping), **`Set varOneNoteResolverResult ExistingMapping`** 🟢 fixed 2026-07-27 — was found to have **no `value` key at all** (contradicting this document's prior ⚪ entry — see correction notice above), causing `Condition Is Genuine Existing Page` to be structurally unreachable via this branch. Now set to `"ExistingMapping"`. See AMEND-2026-07-27-002.

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

### `Condition Should Create Page` ⚪ confirmed clean 2026-07-27 — `equals(outputs('Compose_PageDecision'), 'PAGE_NOT_FOUND')`. False branch (page already exists) traced in full: two `SetVariable` actions there (`Set_varPageAction_ExistsNoCreate`, `Set_varOutputPageSelfUrl_Existing`) were found with empty-string values and live "Invalid parameters" Designer errors — fixed under AMEND-2026-07-27-001 (`ExistsNoCreate` → `"Updated"`; `varOutputPageSelfUrl_Existing` → `variables('varFinalExistingPageSelfUrl')`).

True: **`Create OneNote Page`** ⚪, **`Compose PageSelfUrl Created`** ⚪, **`HTTP Update SP PageSelfUrl`** ⚪
- **`Set varPageAction Created`** ⚪ confirmed clean 2026-07-27 — correctly sets `"Created"` on the genuine fresh-page-creation path (distinct from the mislabeled `UpdatedAppend` sibling below).
- **`Set varOutputPageSelfUrl Created`** 🟢 fixed 2026-06-25 — `outputs('Compose_PageSelfUrl_Created')`
- **`Compose UpdateHtmlFragment`** ⚪, **`Compose ExistingPageId`** ⚪
- **`Set varOutputPageLink Created`** 🟢 confirmed already fixed — `outputs('Create_OneNote_Page')?['body']?['links']?['oneNoteWebUrl']?['href']`

### `Condition Is Genuine Existing Page` 🟢 fixed 2026-07-27 — **this document's prior "⚪ clean" entry was wrong**; the condition was structurally unreachable (checked for literal `'Exists'`, which no action in Flow B ever set — see correction notice above and `2026-07-27-condition-is-genuine-existing-page-defect.md` for the full trail, including a first fix attempt that was itself insufficient). Now: `contains(createArray('ExistingMapping', 'ExistingSection'), variables('varOneNoteResolverResult'))`. Live-verified: evaluates True correctly, routes to the update-existing-page branch, no duplicate page created.
True: **`Get Sections Existing Branch`** ⚪, **`Filter Existing Section By Name`** ⚪, **`Apply to each Existing Section`** ⚪ → **`Update page content Existing Branch`** ⚪ (confirmed actually executing live for the first time 2026-07-27, per the run trace in AMEND-2026-07-27-002)
- **`Set varPageAction UpdatedAppend`** 🟢 fixed 2026-07-27 — was incorrectly set to `"Created"` (this document's prior 🟡 "not confirmed" entry undersold the issue; it was an active mislabeling, not just unverified). Now `"Updated"`. See AMEND-2026-07-27-001.
- **`Set varOutputPageLink Existing`** 🟢 fixed 2026-06-25 — `outputs('Compose_ExistingPageSelfUrl')`; independently re-confirmed present and correct via Peek Code 2026-07-27.
False: **`Create Page OneOff`** ⚪, **`Set varOutputPageLink Created OneOff`** 🟡 — still not confirmed as of 2026-07-27; carried over as an open item.

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
- ~~`Set varPageAction Created`, `Set varPageAction UpdatedAppend`, `Set varOutputPageLink Created OneOff` — not confirmed clean or broken this session~~ — resolved 2026-07-27: `varPageAction Created` confirmed clean, `varPageAction UpdatedAppend` was actually broken and is now fixed (see above). `Set varOutputPageLink Created OneOff` remains unconfirmed, carried forward below.
- `Set varOutputPageLink Created OneOff` (Flow B, one-off fresh-page-creation path) — still not independently verified as of 2026-07-27.
- `Compose IgnoreSeriesMasterId` — literal `''`, low priority, not yet fixed.
- `runAfter` casing — unresolved, see Flow B-wide issues.
- Live UJ1 re-test pending — both flows published 2026-06-25, connections confirmed Connected. Test in a brand new Teams thread.
- UJ2, UJ3, UJ4 — all Flow A fixes now applied; UJ2/UJ3/UJ4 are theoretically unblocked pending UJ1 baseline confirmation.
- **New 2026-07-27**: given this document was shown to record fixes that weren't actually live (see correction notice at top), a broader non-targeted re-verification pass of every `SetVariable` action in both flows (not just the "Existing/Exists"-named ones already checked) is recommended before this document is trusted as fully current again. The targeted "Existing/Exists" naming pattern was swept in full on 2026-07-27 and found clean beyond the three corrections above; the remaining ~40+ `SetVariable` actions across both flows (particularly "...Created"/"...CreatedOneOff" named ones) have not been individually re-verified this pass.
- Flow B's `OutStatus` differentiation (six-state contract, `SUCCESS`/`RECURRING_SETUP_REQUIRED`/`PARTIAL_SUCCESS`/`SETUP_SECTION_NOT_FOUND`/`SETUP_SECTION_AMBIGUOUS`/`ERROR`) remains the top open build item per `2026-07-20-gap-analysis-original-brief-vs-current-build.md`, now with a full structural trace completed in `2026-07-27-flow-b-outstatus-trace.md` ready to build from.
