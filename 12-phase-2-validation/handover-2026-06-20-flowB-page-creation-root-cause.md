# Flow B — Page Link Missing / Duplicate Section Root Cause (2026-06-20)

## Symptom
On every test (recurring AND one-off, e.g. QWE Meeting, TTT Meeting):
- Agent returns success message but `outcreatedpagelink` is always empty.
- TWO OneNote sections get created per meeting: `Mtg - <Title>` and `<Title>` (no prefix).
- Both sections are empty — no page is ever created inside either.
- SharePoint `MeetingNoteIndex` row count stays at 0 in most tests.

## Root cause (confirmed via run-trace tracing this session)

Flow B's one-off path contains **two separate, redundant "find-or-create section" sub-flows** chained one after another, almost certainly left over from a flow rebuild/merge where an old implementation wasn't deleted when a new one was added.

**Block 1 (suffix `_OneOff`):**
`Filter_OneNote_Section_OneOff` → `Compose_Section_Match_Count_OneOff` (`length(body('Filter_OneNote_Section_OneOff'))`) → `Condition Section Exists OneOff` → (False) → `Create Section OneOff` → creates section named **`Mtg - <Title>`** (HTTP 201 confirmed, e.g. section id `1-9914b85d-fb53-4432-a15b-7f0c32a7977f` for TTT Meeting) → `Set varTargetSectionPagesUrl OneOff Created`.

**Block 2 (no suffix, unprefixed naming):**
`Compose SectionDisplayName` → `Compose SafeSectionName` → `Get sections in notebook` → `Filter OneNote Section By Name` → `Compose OneNote Section Match Count` → `Condition OneNote Section Exists` → (False, because block 1's freshly-created section has the `Mtg -` prefix so doesn't match this filter) → `Create OneNote Section` → creates a **second, unprefixed section** named **`<Title>`** → `Get sections in notebook 1` → `For each` / `Filter array` → `VarTargetSectionPagesUrl 2` → `Set varTargetSectionPagesUrl Created` → `Set varOneNoteResolverResult Created`.

**Neither block ever creates an actual OneNote page.** Both only resolve/create a *section* and a *pagesUrl* endpoint. This explains why both sections are always empty.

## Downstream: Condition Should Create Page

After the two blocks converge (and after a `Condition Should Write Mapping` step that writes a SharePoint row), the flow reaches `Condition Should Create Page`:

```
equals( equals(outputs('Compose_PageDecision'), 'PAGE_NOT_FOUND'), true )
```

This evaluates **False** on every test observed so far — including immediately after a section was *just* freshly created in the same run — sending the flow into the "page already exists, don't create" branch (`Set varPageAction ExistsNoCreate` → `Set varOutputPageSelfUrl Existing`, or the equivalent `Set varPageAction UpdatedAppend` / `Set varOutputPageLink Existing` seen in an earlier trace). This is why no page-creation HTTP call ever fires and `varOutputPageLink`/`varOutputPageSelfUrl` is always populated from a non-existent "existing" source — hence always blank.

Earlier in this session, on the **recurring-meeting path**, `Compose_PageDecision` itself was found **Skipped** (`ActionConditionFailed` on its `runAfter`, cascading from `Filter_Existing_Mapping`/`Compose_ExistingPageSelfUrl` also being Skipped) — but that block belongs only to the recurring/SharePoint-mapping branch (gated by `Condition IsRecurring` = True) and is correctly not relevant to one-off meetings like TTT.

For the **one-off path** specifically, the input to `Condition Should Create Page` (`Compose_PageDecision`, or whatever its one-off equivalent is — needs to be located in Designer) is returning `PAGE_EXISTS`-equivalent even when nothing exists yet. The likely cause is that whatever composes this decision is reading a "does it exist" signal set too early (e.g. before either section-creation block ran), or is reading from `varOneNoteResolverResult`/`varTargetSectionPagesUrl` values that were initialised with a default of "Exists" rather than correctly reset to "CreateRequired" once it's confirmed the section is brand new.

## Next steps (not yet done)
1. Locate the exact Compose action that feeds `Condition Should Create Page` on the **one-off** path (may not be `Compose_PageDecision` — that name appeared on the recurring path; the one-off path may have its own equivalent, possibly built from `varOneNoteResolverResult` / `varPageAction`). Open its Code view to see the expression and trace what variable it reads.
2. Decide remediation for the duplicate section-creation blocks: the cleanest fix is almost certainly to **delete Block 2 entirely** (the unsuffixed `Compose SectionDisplayName` → ... → `Set varOneNoteResolverResult Created` chain) since Block 1 (`_OneOff` suffixed) already does the same job correctly with the `Mtg -` prefix that matches the established section-naming convention. This needs confirming against `01-shared-contract/shared-journey-contract-vfinal.md` to check which naming convention (`Mtg - <Title>` vs `<Title>`) is actually the documented/intended one.
3. After removing the duplicate block, add the missing **OneNote "Create page" action** (using `PageHtml`/`Concatenate(...)` content already fixed earlier this session) inside the True branch of `Condition Should Create Page`, since neither surviving block currently creates a page at all.
4. Re-test TTT Meeting (clean slate — delete both sections + any SharePoint row first) and confirm: one section only, a real page inside it, and a populated page link in the agent's response.
5. Once one-off path confirmed working end-to-end, return to FA12 `isrecurring`/`seriesmasterid` fix (drafted, not applied) to unblock UJ3/UJ4 recurring-path testing, since the recurring branch will likely need the same "actually create the page" fix applied to its own equivalent of `Condition Should Create Page`.

## Session test record
- TTT Meeting (genuine one-off, confirmed in Outlook): success message returned, link missing, two sections created (`Mtg - TTT Meeting`, `TTT Meeting`), both confirmed empty by user.
- Confirms bug is independent of recurrence/FA12; reproduces on the clean one-off path.
