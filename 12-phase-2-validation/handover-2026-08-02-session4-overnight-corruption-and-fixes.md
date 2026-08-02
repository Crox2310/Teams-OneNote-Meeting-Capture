# Handover — 2 August 2026 (Session 4, overnight into afternoon) — Corruption recurrence, four new bugs fixed, one logged open

## ⏭ START HERE NEXT SESSION

**Status: Flow B published and live as of ~13:40. Three genuine new bugs found and fixed this session (beyond the corruption saga). One further genuine bug found and confirmed but NOT yet fixed — see "OPEN — sectionId scope bug" below, this is the next priority.** Also outstanding: draft the Microsoft support ticket (evidence is now excellent — see "Support ticket evidence" section).

This session continued directly from `handover-2026-08-01-session3-continuation.md`. That doc left off with `OF09b` broken mid-repair. This session: rebuilt `OF09b` from scratch, published, then ran the three post-publish test scenarios — which surfaced multiple genuine latent bugs never caught by any prior static review, on top of at least three further live occurrences of the value-corruption phenomenon first documented in `handover-2026-08-01-corruption-incident-and-fix-list.md`.

---

## Part 1 — OF09b rebuild (carried over from session 3)

Confirmed `OF09b — HTTP Update SP PageSelfUrl (OneOff)` had been left broken (mid-repair, missing `runAfter`, malformed Uri) at the end of session 3. Rebuilt from scratch character-by-character, sourcing the em dash live from `OF02`'s Code view rather than typing/pasting it. Final correct Uri:

```
_api/web/lists/GetByTitle('RecurringMeetingSectionMap')/items(@{if(greater(length(body('OF01—Filter_Existing_Mapping_OneOff')),0), first(body('OF01—Filter_Existing_Mapping_OneOff'))?['ID'], body('OF09a—Send_an_HTTP_request_to_SharePoint_(OneOff)')?['ID'])})
```

Two additional bugs found and fixed during this rebuild, not previously documented:

**TOPFIX01 — `varFinalPageDecision` top-level initializer had a genuine logic error.** It was set to `if(not(empty(outputs('Compose_PageSelfUrl_Created'))), 'PAGE_EXISTS', 'PAGE_NOT_FOUND')` — but `Compose_PageSelfUrl_Created` sits deep inside a downstream branch that hasn't run at the point this top-level initializer executes, which is structurally impossible and blocked every Publish/Save with a `BadRequest`/`InvalidTemplate` error. Original fix-list doc's own fix #2 appears to have introduced this. Corrected to blank (matching the "confirmed correctly blank-by-design" pattern documented for its sibling top-level variables). **This variable has since reverted to the broken expression at least three further times this session — see corruption section below.**

**Self-reference bug — `Set_varTargetSectionPagesUrl_ExistingMapping` could not be published.** Original documented value (fix #15 in the 1 Aug fix-list) was:
```
if(equals(toLower(string(triggerBody()?['text'])), 'true'), first(body('Filter_Existing_Mapping'))?['SectionPagesUrl'], variables('varTargetSectionPagesUrl'))
```
This has never actually been publishable — Power Automate rejects a `SetVariable` action whose value expression references its own variable's current value ("Self reference is not supported when updating the value of variable"). This must have always failed at Publish time; nobody had gotten this far before. Fixed properly (**Option A**, chosen over a quick blank-string workaround after the workaround caused a live regression — see below) by restructuring:

- Deleted the single `SetVariable` action.
- Replaced with a new `Condition_Recurring_TargetSection` (`equals(toLower(string(triggerBody()?['text'])), 'true')` = `true`).
- True branch: `Set_varTargetSectionPagesUrl_ExistingMapping`, simplified value `first(body('Filter_Existing_Mapping'))?['SectionPagesUrl']` (no self-reference needed now the branch guarantees this only runs when recurring).
- False branch: left completely empty (0 actions) — this is what correctly leaves the variable untouched on the one-off path, which a `SetVariable`-with-blank-fallback cannot do.

(A quicker fix — blanking the else-value to `''` instead of restructuring — was tried first and caused a live regression: `Create_Page_OneOff`'s `sectionId` parameter depends on `varTargetSectionPagesUrl` retaining its one-off-path value, and blanking it broke page creation for the "existing mapping row was stale" one-off sub-case. Reverted and rebuilt properly as above.)

---

## Part 2 — Corruption recurrence (three further occurrences, on top of the original 1 Aug incident)

**This is now a well-evidenced, repeatable pattern across two separate days.** Summary for the support ticket:

- **Occurrence 2 (this session, early):** `varFinalPageDecision` reverted to the exact broken TOPFIX01 expression, blocking Publish with the identical BadRequest error, despite having been fixed and saved cleanly earlier. Turned out on inspection to be a false alarm on the second sighting (already blank) but genuinely reverted on a third sighting minutes later.
- **Occurrence 3 (this session):** After restoring to a known-clean, previously-**Published** version (12:24) specifically to escape a 26-action blanking event, making **exactly one small edit** (fixing a single blank value) and saving triggered a fresh, near-total corruption — **26 actions blanked simultaneously**, matching the same signature as the original 1 August incident almost exactly (same set of affected actions, largely).
- Full list of the 26 affected actions and their correct target values is preserved in this session's chat log and was worked through top-to-bottom a second time; all confirmed restored and spot-checked (`Set_varOutputPageLink_Created_OneOff_Gate`, `varFinalPageDecision 1`, and the new `Condition_Recurring_TargetSection`/`Set_varTargetSectionPagesUrl_ExistingMapping` pair) before the most recent Publish.

**Root-cause observations, refined this session (useful for the support ticket):**
1. **Only ever `SetVariable`/`InitializeVariable` `value` fields are affected** — never `Compose`, HTTP actions, Condition expressions, headers, or connector parameters. A very narrow, specific signature.
2. **Not confined to recently-edited actions** — corruption hits actions untouched for hours alongside ones just built minutes earlier, in the same event.
3. **Timing correlates with Save**, not idle time this session — a single small edit + save on a known-good restored version triggered full corruption within minutes. This is harder to reconcile with the original incident doc's Theory B (idle/environmental) and leans the evidence more towards something in the save/reserialization path itself.
4. The Copilot Studio UI showed an *"Enjoying Express mode?"* prompt during at least one affected session — **not yet confirmed whether this flow is running in Power Automate "Express" design mode**; worth checking, as it would sharpen the support ticket considerably if so.

**Support ticket:** not yet drafted. Still on the list. Given the strength of this session's evidence (single edit → total corruption, on a version that had *just* been confirmed clean and published), this should be a priority next step — arguably before further flow editing resumes, since a support response could change the whole approach.

---

## Part 3 — Genuine new bugs found via live testing (post-publish)

Running the three standard test scenarios against the newly-published flow surfaced **four further genuine, previously-undetected bugs** — none caught by any prior static Code-view review, all only visible once the specific code path was actually exercised end-to-end.

### Bug 1 — FIXED: `OF05c` type mismatch (String vs Integer)
`OF05c — Set varFinalMatchCount (OneOff)` set `varFinalMatchCount` (declared type String) directly from `outputs('OF04_—_Compose_Match_Count_OneOff')`, which evaluates to an Integer at runtime. Power Automate enforces type strictly on declared-type variables. Compare with the working recurring-branch equivalent (fix #5, `varFinalMatchCount_1`), which correctly wraps in `string(...)`. This bug has been sitting in the original fix-list doc's own documented value for `OF05c` (fix #14) since 1 August and never been exercised until this session's live one-off test.

**Fix applied:** `value: string(outputs('OF04_—_Compose_Match_Count_OneOff'))`.

### Bug 2 — FIXED: missing `varOutputPageLink`/`varOutputPageSelfUrl`/`varPageAction` on the one-off "created" path
`OF09-Gate`'s False (one-off) branch only ever wrote to the SharePoint mapping table (`OF09b-i`, `OF09b`) — it never set `varOutputPageLink`, `varOutputPageSelfUrl`, or `varPageAction`. These are only set on the True (recurring) branch, via `Set_varPageAction_Created`, `Set_varOutputPageSelfUrl_Created`, `Set_varOutputPageLink_Created`. Result: a brand-new one-off meeting successfully created its OneNote page and wrote the SharePoint mapping, but the Teams success message came back with **no page link** — `Topic.OutCreatedPageLink` was empty.

**Fix applied:** three new actions added inside `OF09-Gate`'s False branch, after `OF09b`, mirroring the True-branch pattern:
- `Set_varPageAction_Created_OneOff` → `varPageAction` = `Created`
- `Set_varOutputPageSelfUrl_Created_OneOff` → `varOutputPageSelfUrl` = `outputs('Compose_PageSelfUrl_Created')`
- `Set_varOutputPageLink_Created_OneOff_Gate` → `varOutputPageLink` = `outputs('Create_OneNote_Page')?['body']?['links']?['oneNoteWebUrl']?['href']`

(Named with a `_Gate` suffix to disambiguate from a pre-existing, differently-scoped action of a very similar name and purpose already present elsewhere in the flow — `Set_varOutputPageLink_Created_OneOff`, inside `Condition_Is_Genuine_Existing_Page`'s False branch, which handles a different sub-case: a one-off meeting whose existing-mapping page turned out to be stale/invalid and needed recreating. These are legitimately two separate code paths needing the same kind of fix, not a duplicate.)

### Bug 3 — FIXED: self-reference block on `Set_varTargetSectionPagesUrl_ExistingMapping`
Covered in Part 1 above (Option A, Condition-wrap restructure).

### Bug 4 — OPEN, confirmed, NOT YET FIXED: `sectionId` scope error on `Create_OneNote_Page`
Most recent test run (13:39) failed with: *"Flow run failed. Action 'Create_OneNote_Page' failed: The section id given in the input is invalid."*

`Create_OneNote_Page` (the True/recurring-path page-creation action, positioned before `OF09-Gate`) has:
```
sectionId: items('Apply_to_each')?['pagesUrl']
```
`items('Apply_to_each')` is only valid **while executing inside that specific loop**. `Create_OneNote_Page` runs outside and after any `Apply_to_each` loop closes — this is out-of-scope dynamic content, almost certainly inserted via the dynamic-content picker at some point while the canvas was positioned near the loop, and never actually exercised until this session's live recurring-path test.

**Recommended fix (not yet applied):** change to `variables('varTargetSectionPagesUrl')` — the variable that appears to be the intended carrier of this value into later actions on this path, based on the flow's naming convention and how the equivalent one-off actions are structured. **Needs confirming against a live trace before applying** — do not assume without checking, given tonight's track record of assumptions needing correction.

---

## Test scenario status

- **New one-off meeting**: ultimately passed after Bug 1 and Bug 2 fixes (page created, mapping written, link now present in success message). Re-run once more after the corruption/restore cycle to confirm still passing — **not yet re-confirmed post the most recent Publish**, do this first next session.
- **Recapture same one-off meeting**: not yet run this session.
- **Recurring regression test**: attempted, surfaced Bug 4 (`sectionId` scope error) — **currently failing**. Do not consider recurring path validated until this is fixed and re-tested.

---

## Recommended order for next session

1. Re-confirm current published state is genuinely stable: fresh Flow Checker pass, spot-check `Set_varOutputPageLink_Created_OneOff_Gate`, `varFinalPageDecision` (top-level and `_1`), and `Condition_Recurring_TargetSection` via Code view before touching anything else.
2. Fix Bug 4 (`sectionId` scope error) — confirm via live trace what the correct source of the section-pages-URL value is at that point in the recurring path before editing.
3. Re-run all three test scenarios cleanly end-to-end, in fresh Teams threads, checking outputs (not just "no error") each time.
4. Draft and submit the Microsoft support ticket for the corruption pattern — evidence is now strong (single-edit-triggers-total-corruption on a freshly-published clean version).
5. Once stable: revisit low-priority items — `Get items` OData filter/top-parameter warning, `Compose_AgentResponseSummary` cosmetic defect (falls through to generic message; documented in the 1 Aug fix-list, never actioned), the six-value `OutStatus` differentiation (still tracked separately, unrelated to this session).

## Status

**Published and live. Two of three test scenarios exercised; recurring path currently broken (Bug 4, diagnosed, fix not yet applied). Corruption pattern recurred at least twice more this session — strong, ticket-ready evidence now exists. Support ticket still to be written.**
