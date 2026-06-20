# Flow B — Page Link Missing / Duplicate Section Root Cause (2026-06-20)

## Symptom
On every test (recurring AND one-off, e.g. QWE Meeting, TTT Meeting):
- Agent returns success message but `outcreatedpagelink` is always empty.
- TWO OneNote sections get created per meeting: `Mtg - <Title>` and `<Title>` (no prefix).
- Both sections are empty — no page is ever created inside either.
- SharePoint `MeetingNoteIndex` row count stays at 0 in most tests.

## Root cause #1: duplicate section-creation blocks

Flow B's one-off path contains **two separate, redundant "find-or-create section" sub-flows** chained one after another, almost certainly left over from a flow rebuild/merge where an old implementation wasn't deleted when a new one was added.

**Block 1 (suffix `_OneOff`):**
`Filter_OneNote_Section_OneOff` → `Compose_Section_Match_Count_OneOff` (`length(body('Filter_OneNote_Section_OneOff'))`) → `Condition Section Exists OneOff` → (False) → `Create Section OneOff` → creates section named **`Mtg - <Title>`** (HTTP 201 confirmed, e.g. section id `1-9914b85d-fb53-4432-a15b-7f0c32a7977f` for TTT Meeting) → `Set varTargetSectionPagesUrl OneOff Created`.

**Block 2 (no suffix, unprefixed naming):**
`Compose SectionDisplayName` → `Compose SafeSectionName` → `Get sections in notebook` → `Filter OneNote Section By Name` → `Compose OneNote Section Match Count` → `Condition OneNote Section Exists` → (False, because block 1's freshly-created section has the `Mtg -` prefix so doesn't match this filter) → `Create OneNote Section` → creates a **second, unprefixed section** named **`<Title>`** → `Get sections in notebook 1` → `For each` / `Filter array` → `VarTargetSectionPagesUrl 2` → `Set varTargetSectionPagesUrl Created` → `Set varOneNoteResolverResult Created`.

Neither block creates an actual OneNote page on its own — both only resolve/create a *section* and a *pagesUrl* endpoint. `Block 2` is the one that finally sets `varOneNoteResolverResult` to a "CreateRequired"/"Created"-type value used downstream.

## Root cause #2 (the actual page-creation gap, confirmed)

After the two blocks converge (and after a `Condition Should Write Mapping` step that writes a SharePoint row), the flow reaches **`Condition Should Create Page`**, which contains a nested condition **`Condition Is Genuine Existing Page`**:

```
equals(variables('varOneNoteResolverResult'), 'Exists')
```

Confirmed via run trace on a fresh TTT Meeting test: this evaluates **False** (correctly — `varOneNoteResolverResult` was `CreateRequired`/`Created`, not `Exists`, since the section was brand new).

**The branches are inverted / incomplete:**
- **True branch** ("it IS a genuine existing page") contains the full update-page logic: `Get Sections Existing Branch` → `Filter Existing Section By Name` → `Apply to each Existing Section` → `Update page content Existing Branch` → `Set varPageAction UpdatedAppend` → `Set varOutputPageLink Existing`.
- **False branch** ("it is NOT an existing page, i.e. needs creating") contains **"No Actions"** — completely empty.

So on every test where a section/page is genuinely new (which is every test so far), the condition correctly identifies "this is not an existing page" (False), correctly skips the "update existing page" logic — and then does **nothing**, because no one ever built a "create new page" action in the False branch. No page gets created, no `varOutputPageLink`/`varOutputPageSelfUrl` ever gets set on this path, hence the link is always empty in the final agent response.

This is the precise, narrow fix needed: **add a "Create page" (OneNote) action into the False branch of `Condition Is Genuine Existing Page`**, using the section's `pagesUrl` (from `varTargetSectionPagesUrl`/`varTargetSectionPagesUrl Created`) and the `PageHtml` content (built via `Concatenate(...)`, already fixed earlier this session in `C8B_Call_FlowB_Create_Page`/`C10_Call_FlowB_Create_Page`'s caller inputs — need to confirm Flow B receives and threads this through to the new action), then set `varOutputPageLink`/`varOutputPageSelfUrl` from the new page's returned `links.oneNoteWebUrl.href` (or equivalent self URL).

## Recurring-path note (context only, not actioned this session)
Earlier in this session, on the **recurring-meeting path** (gated by `Condition IsRecurring` = True), the equivalent action `Compose_PageDecision` was found **Skipped** (`ActionConditionFailed` cascading from `Filter_Existing_Mapping`/`Compose_ExistingPageSelfUrl` also Skipped). That's a separate sub-flow from the one-off path described above and wasn't reached in this session's testing (TTT Meeting is one-off). It will likely need its own equivalent "create page" action added once UJ3/UJ4 testing resumes (see Next Steps).

## Next steps
1. **Primary fix:** In Designer, open `Condition Is Genuine Existing Page`'s **False** branch and add a OneNote "Create page" action:
   - Site/notebook context: same as used in `Create Section OneOff` / `Create OneNote Section`.
   - Target: `pagesUrl` from `varTargetSectionPagesUrl` (need to confirm which of the two — Block 1's `OneOff Created` or Block 2's `Created` — is the one actually in scope here; likely Block 2's since it ran later, but this should be verified, and ties into item 2 below).
   - Page content: `PageHtml` parameter received from the calling Topic action (`C8B_Call_FlowB_Create_Page` / `C10_Call_FlowB_Create_Page`), already fixed to use `Concatenate(...)` earlier this session.
   - After creation, add a `Set varOutputPageLink`/`Set varOutputPageSelfUrl` (Created) action using the new page's URL from the HTTP response, mirroring `Set varOutputPageLink Existing` in the True branch.
2. **Secondary fix (duplication cleanup):** Decide on remediation for the duplicate section-creation blocks. Cleanest approach is almost certainly to **delete Block 2 entirely** since Block 1 (`_OneOff` suffixed) already creates the section correctly with the `Mtg -` prefix that matches the established naming convention — confirm convention against `01-shared-contract/shared-journey-contract-vfinal.md` first. Note: if Block 2 is deleted, `varOneNoteResolverResult` is currently only set by Block 2 (`Set varOneNoteResolverResult Created`/`Exists`) — Block 1 will need to set this variable itself (or an equivalent) so `Condition Is Genuine Existing Page` still has a valid input to check.
3. Re-test TTT Meeting (clean slate — delete both sections + any SharePoint row first) and confirm: one section only, a real page inside it, and a populated page link in the agent's response.
4. Once one-off path confirmed working end-to-end, return to FA12 `isrecurring`/`seriesmasterid` fix (drafted, not applied) to unblock UJ3/UJ4 recurring-path testing — the recurring branch will need the analogous "actually create the page" fix applied once the `Compose_PageDecision` Skipped/runAfter chain on that path is also resolved.

## Session test record
- TTT Meeting (genuine one-off, confirmed in Outlook): success message returned, link missing, two sections created (`Mtg - TTT Meeting`, `TTT Meeting`), both confirmed empty by user.
- Confirms bug is independent of recurrence/FA12; reproduces on the clean one-off path.
- Root cause fully isolated to: (1) duplicate section-creation blocks, (2) `Condition Is Genuine Existing Page` False branch has no page-creation action.

## RESOLUTION (confirmed same session, 2026-06-20)

Both root causes fixed and verified via live Teams/Copilot Studio test on a clean TTT Meeting:

**Fix 1 — Page creation (Root cause #2):** Added a new OneNote "Create page in a section" action (named `Create Page OneOff`) into the previously-empty False branch of `Condition Is Genuine Existing Page`. Configured with:
- Notebook Key: `Meeting Notes|$|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Meeting Notes`
- Notebook section: `variables('varTargetSectionPagesUrl')`
- Page Content: `PageHtml` (the calling Topic action's input parameter)

Followed by a new `Set variable` action (`Set varOutputPageLink Created OneOff`) setting `varOutputPageLink` to:
```
outputs('Create_Page_OneOff')?['body']?['links']?['oneNoteWebUrl']?['href']
```

**Fix 2 — Duplicate sections (Root cause #1):** Deleted Block 2 entirely (`Compose SectionDisplayName` through `Set varOneNoteResolverResult Created`, including the nested `Condition OneNote Section Exists` and its branches). To preserve the `varOneNoteResolverResult` variable that downstream logic (`Condition Is Genuine Existing Page`) depends on, added two new `Set variable` actions to Block 1 (`Condition Section Exists OneOff`):
- True branch (section already exists): `varOneNoteResolverResult` = `Exists`
- False branch (section just created): `varOneNoteResolverResult` = `Created`

**Complication found mid-fix:** Deleting Block 2 initially broke `Filter_Existing_Section_By_Name` (in the True/"update existing page" branch of `Condition Is Genuine Existing Page`), which depends on `Compose_SafeSectionName`'s output (`"where": "@equals(item()?['name'],outputs('Compose_SafeSectionName'))"`). `Compose SectionDisplayName` and `Compose SafeSectionName` were NOT pure Block 2 cruft — they're a shared dependency also used by the existing-page-update logic. Restored both actions (copied back in from a previously saved version of the flow) positioned after `Compose Branch Result NoMatch`, before `Condition Should Write Mapping`, so both the True branch's filter and Block 1 can resolve correctly. Error cleared after restoring; flow saved successfully.

**Verification test (TTT Meeting, clean slate — both old sections deleted first):**
- OneNote: exactly ONE section created (`Mtg - TTT Meeting`), containing one real page ("Untitled Page") with correct H1 + paragraph content matching the `PageHtml` input. No duplicate unprefixed section created.
- Agent response: returned a populated, working page link (`...Mtg%20-%20TTT%20Meeting.one...Untitled%20Page...`).
- SharePoint: no row created (correct — TTT Meeting is one-off, mapping is recurring-only).

## Status: RESOLVED for one-off path (UJ1/UJ2-adjacent one-off creation flow)

## Remaining work (separate from this fix)
- Recurring-meeting path (`Condition IsRecurring` = True) was NOT touched this session. It has its own equivalent "create page" logic (`Compose_PageDecision`, `Condition Should Create Page`'s True branch with `Create OneNote Page`) which was previously found with a Skipped/runAfter cascade issue (`Compose_PageDecision` → `Compose_ExistingPageSelfUrl` → `Filter_Existing_Mapping` all Skipped on a recurring test). This needs its own investigation once FA12 `isrecurring`/`seriesmasterid` fix (drafted, not applied) is in place to properly route recurring meetings for UJ3/UJ4 testing.
- Flow B has NOT yet been published (still saved/draft) — confirm with David before publishing, or check if it was published as part of this session's later steps.
- `candidatelist: "''"` cosmetic bug in Flow A (parked, separate issue).
