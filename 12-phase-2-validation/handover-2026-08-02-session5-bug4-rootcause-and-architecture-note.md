# Handover — 2 August 2026 (Session 5, afternoon continuation) — Bug 4 root cause found, fresh corruption event, architecture recommendation

## ⏭ START HERE NEXT SESSION

**Status: Flow currently in a 26-error blanked state (fresh corruption event, occurred live during this session with a witnessed 0→26 flip in roughly 10 seconds). Not published in this state — do not let anyone use it until fixed.** The 26-item fix list from session 4 is still accurate and reusable (see below, reprinted with one caveat). Once fixed, re-check `Set_varTargetSectionPagesUrl_OneOff_Exists` (item 19) specifically, since its correctness now depends on a structural fix made this session (see Bug 4).

This session continued directly from `handover-2026-08-02-session4-overnight-corruption-and-fixes.md`.

---

## Bug 4 — actually two separate bugs, both now understood; one fixed, one needs re-verification

Session 4 left Bug 4 open as a single suspected issue (`sectionId` referencing out-of-scope loop content on `Create_OneNote_Page`). This session's investigation found it's really **two distinct problems** on two different code paths that both feed into the same downstream `sectionId` failure symptom.

### Bug 4a — FIXED: `For each 1` iterating over the wrong (unfiltered) collection

Located inside `Condition Section Exists OneOff` → True branch, in the one-off "existing section found" path. Its `foreach` was:
```
outputs('Get_Sections_OneOff')?['body/value']
```
— i.e. **every section in the entire notebook** (confirmed via live trace: 4 sections returned, including unrelated ones like "Play Book" and "Recurring Meetings"), not the single, correctly-filtered result from `Filter_OneNote_Section_OneOff` (confirmed via live trace to correctly return exactly 1 matching section). The filter action ran and worked; its output was simply never wired into the loop that consumes it downstream.

Effect: `Set_varTargetSectionPagesUrl_OneOff_Exists` (`items('For_each_1')?['pagesUrl']`) could capture the `pagesUrl` of *any* section in the notebook depending on iteration order/index, not necessarily the meeting's actual section. Worked by coincidence when the correct section happened to sort first; would silently pick a wrong section otherwise. This is the true root cause of the "section id given in the input is invalid" failures seen against one-off "existing section" meetings.

**Fix applied:**
```
foreach: body('Filter_OneNote_Section_OneOff')
```
(Matching the pattern already used correctly in the recurring branch's equivalent loop and in `Apply_to_each_Existing_Section`.)

**Confirmed correct via Code view before the fresh corruption event hit (see below) — needs re-confirming this session's fix survived, since it may be one of the 26 blanked items or structurally reverted.**

### Bug 4b — NOT a bug: `Create_OneNote_Page`'s `sectionId` was already correct

Contrary to session 4's suspicion, `Create_OneNote_Page` (the True/recurring-path page-creation action, before `OF09-Gate`) was checked directly this session and reads:
```
sectionId: variables('varTargetSectionPagesUrl')
```
— already correct, no `items('Apply_to_each')` reference. **A live recurring-meeting test this session succeeded end-to-end** (page created, link returned, worked in Teams) — first clean recurring-path pass this session, confirming this action is fine.

**Important caveat discovered:** a *separate*, stale failed run (timestamped 3:10 PM) was initially mistaken for a live/current failure of this same action, and briefly reopened the Bug 4b investigation unnecessarily. That old run's raw inputs showed the pre-fix `items('Apply_to_each')?['pagesUrl']` expression — evidence that a run's inputs reflect what was *published* at the time it executed, not the current draft. **Lesson for future sessions: always check the run's timestamp against the most recent Publish time before treating a failed run as evidence of the current flow's state.**

---

## Fresh corruption event — Occurrence 4/5 (numbering continues from session 4's doc)

Mid-session, David ran Flow Checker twice in quick succession: **0 errors**, then **26 errors**, roughly **10 seconds apart**, with no edit made in between. This is the cleanest, most precisely-timed evidence of the corruption phenomenon captured so far across all sessions — a live, witnessed flip rather than something discovered after the fact.

The 26 affected actions are, once again, the same set as the original 1 August incident and session 4's occurrence 3 — same signature (`SetVariable`/`InitializeVariable` `value` fields only). Full list and correct target values below, unchanged from session 4's record, reprinted here for continuity:

| # | Action | Correct value |
|---|---|---|
| 1 | `varTargetSectionPagesUrl 1` | `items('Apply_to_each')?['pagesUrl']` |
| 2 | `varOneNoteResolverResult 1` | `ExistingSection` |
| 3 | `varTargetSectionPagesUrl 2` | `outputs('Create_Section_Recurring')?['body']?['pagesUrl']` |
| 4 | `varOneNoteResolverResult 2` | `CreatedSection` |
| 5 | `Set varPageAction Created` | `Created` |
| 6 | `Set varOutputPageSelfUrl Created` | `outputs('Compose_PageSelfUrl_Created')` |
| 7 | `Set varOutputPageLink Created` | `outputs('Create_OneNote_Page')?['body']?['links']?['oneNoteWebUrl']?['href']` |
| 8 | `Set varPageAction Created OneOff` | `Created` |
| 9 | `Set varOutputPageSelfUrl Created OneOff` | `outputs('Compose_PageSelfUrl_Created')` |
| 10 | `Set varOutputPageLink Created OneOff Gate` | `outputs('Create_OneNote_Page')?['body']?['links']?['oneNoteWebUrl']?['href']` |
| 11 | `Set varPageAction ExistsNoCreate` | `Updated` |
| 12 | `Set varOutputPageSelfUrl Existing` | `variables('varFinalExistingPageSelfUrl')` |
| 13 | `Set varPageAction UpdatedAppend` | `Updated` |
| 14 | `Set varOutputPageLink Existing` | `variables('varFinalExistingPageSelfUrl')` |
| 15 | `Set varOutputPageLink Created OneOff` | `outputs('Create_Page_OneOff')?['body']?['links']?['oneNoteWebUrl']?['href']` |
| 16 | `varFinalExistingPageSelfUrl 1` | `outputs('Compose_ExistingPageSelfUrl')` |
| 17 | `varFinalPageDecision 1` | `outputs('Compose_PageDecision')` |
| 18 | `varFinalMatchCount 1` | `string(outputs('Compose_Match_Count'))` |
| 19 | `Set varTargetSectionPagesUrl OneOff Exists` | `items('For_each_1')?['pagesUrl']` — **re-verify `For_each_1`'s `foreach` itself also survived (Bug 4a fix)** |
| 20 | `Set varOneNoteResolverResult Exists OneOff` | `ExistingSection` |
| 21 | `Set varTargetSectionPagesUrl OneOff Created` | `outputs('Create_Section_OneOff')?['body']?['pagesUrl']` |
| 22 | `Set varOneNoteResolverResult Created OneOff` | `CreatedSection` |
| 23 | `OF05a — Set varFinalExistingPageSelfUrl (OneOff)` | `outputs('OF02_—_Compose_ExistingPageSelfUrl_OneOff')` |
| 24 | `OF05b — Set varFinalPageDecision (OneOff)` | `outputs('OF03_—_Compose_PageDecision_OneOff')` |
| 25 | `OF05c — Set varFinalMatchCount (OneOff)` | `string(outputs('OF04_—_Compose_Match_Count_OneOff'))` |
| 26 | `Set varOutStatus` | `OK` |

**This fix pass had not been completed when this session ended.** Do this first next session, then Flow Checker, then Publish immediately (minimal delay, per the pattern that's worked best in prior sessions), then re-run all three test scenarios fresh.

---

## Architecture recommendation — raised by David, worth acting on

David raised a reasonable question: whether the flow's size/nesting depth is a contributing factor to the corruption frequency and severity, even if not the root cause (which increasingly looks like a genuine Microsoft platform bug — see prior sessions' evidence, e.g. the devhut.net report describing an near-identical symptom on a description that doesn't suggest an unusually large flow).

**Assessment:** size is unlikely to be the *root cause* (external evidence of the same bug on other people's flows exists), but it plausibly amplifies **frequency of exposure** (bigger document to reserialize on every save = more chances to hit whatever bug this is) and **blast radius per event** (more eligible `SetVariable`/`InitializeVariable` actions = more get hit each time it fires) and **diagnosis time** (more nesting to navigate before finding the actually-affected action).

**Recommendation for a future, calm (non-firefighting) session:** split Flow B into two child flows — one for the recurring path, one for the one-off path — called from a lightweight parent flow that only handles `Condition_IsRecurring`-equivalent routing. This is a natural seam that already exists in the current logic. Benefits: roughly halves the blast radius of any single future corruption event, and meaningfully shortens the navigation/diagnosis path when something does go wrong. Does not require waiting on Microsoft support — can be done independently. Not urgent, not to be attempted mid-incident.

---

## Recommended order for next session

1. Confirm current flow state (expect 26 errors per above, unless someone has since fixed them) — do NOT assume clean without checking.
2. Work through the 26-item fix list top to bottom, saving as you go.
3. Specifically re-verify Bug 4a's fix (`For_each_1`'s `foreach` = `body('Filter_OneNote_Section_OneOff')`) survived — this is new since session 4 and hasn't been battle-tested against a corruption cycle yet.
4. Flow Checker → Publish immediately once clean, no delay.
5. Re-run all three test scenarios fresh (new one-off, recapture one-off, recurring) in new Teams threads — recurring is now believed genuinely fixed (passed live this session) but has not survived a full corruption-recovery cycle yet, so don't skip re-testing it.
6. Draft and submit the Microsoft support ticket — evidence is now very strong, including a precisely-timed 0→26-errors-in-10-seconds live observation from this session. This remains the single most important non-code action outstanding across all recent sessions.
7. Once stable, and in a separate, unhurried session: scope the two-child-flow architecture split described above.
8. Low-priority, unrelated to any of the above: `Get items` OData filter warning, `Compose_AgentResponseSummary` cosmetic defect, six-value `OutStatus` differentiation.

## Status

**Flow in a broken (26-error) state at session end — not published. Bug 4 fully diagnosed: one part (Bug 4a, `For_each_1` scope) fixed but not yet re-verified post-corruption; the other part (Bug 4b) turned out not to be a bug at all. Recurring path confirmed working end-to-end via live test this session — a genuine, hard-won milestone. Support ticket still not drafted despite now having excellent evidence; this should be treated as high priority.**
