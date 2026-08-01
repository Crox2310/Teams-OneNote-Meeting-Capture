# Handover — 1 August 2026: Draft Corruption Incident — Diagnosis and Full Fix List

## ⏭ START HERE NEXT SESSION

**Status: fix list complete and verified against the 31 July known-good archive. Fixes not yet applied to the live draft as of this doc being written.** This session (following on from `handover-2026-08-01-oneoff-build-session.md`) discovered that Flow B's draft had suffered **widespread, silent value corruption** — many `SetVariable`/`InitializeVariable` action values, and several `body()`/`outputs()` expression references, had gone blank or been altered without any deliberate edit from David or Claude. This doc is the complete record of what was found and the exact fix for each.

**Next session should be a pure apply-and-verify session:**
1. Apply all 26 fixes below, in the order listed (canvas top-to-bottom).
2. After each fix, save, then move to the next — do **not** batch-save many fixes and check once at the end; earlier in this session that pattern coincided with fixes appearing to "stick" only briefly before reverting.
3. Once all 26 are applied, run **Flow Checker** and confirm **Errors (0)**. Do not treat a lower-but-nonzero count as acceptable — go back through the Errors list exhaustively.
4. **Critically: before publishing, re-open and re-verify (via Code view) every one of the 26 fixed actions one more time**, even if Flow Checker shows 0 errors. This session found that Flow Checker's error list did not always reflect every corrupted action, and some previously-fixed actions (notably OF07, OF08, and the `Condition_Should_Create_Page` false-branch `SetVariable`s) were found reverted to blank on more than one occasion after appearing fixed.
5. Only once a **fresh, independent Code view check of all 26 actions confirms they're correct** should you Publish.
6. Immediately after publishing, run the three test scenarios from `handover-2026-08-01-oneoff-build-session.md` (new one-off meeting, recapture same one-off meeting, recurring regression test).

---

## What happened (as best understood)

Partway through today's build session (documented in `handover-2026-08-01-oneoff-build-session.md`), a Flow Checker pass that had shown **0 errors** was followed, a few minutes later, by a pass showing **4 errors**, then **19 errors** — the blank values appeared on actions that had not been touched in that window, including actions confirmed clean earlier in the same session and actions from **outside** today's build entirely (e.g. the top-level `varFinal*` initializers, `Condition_Section_Exists_Recurring`'s `SetVariable` actions).

Attempts to manually re-patch individual actions saw some of them (notably `OF07`/`Set_varOutputPageLink_Existing`, `OF08`/`Set_varOutputPageSelfUrl_Existing`, and the `Condition_Should_Create_Page` false-branch actions) **revert to blank again after being fixed and confirmed via Code view**, on more than one occasion.

**We do not have a confirmed root cause.** Two working theories, neither fully confirmed:
- **Theory A — edit-triggered:** corruption is a side effect of specific Designer operations performed today, particularly the drag-move-then-delete-rebuild sequence used when constructing `OF09-Gate`'s True branch, or repeated saves in quick succession to nearby actions.
- **Theory B — environmental/platform-level:** the error count grew even during a ~90 minute window (08:41–10:19) when no edits were being made, which is inconsistent with Theory A alone and suggests something outside manual UI edits (a sync/autosave/platform issue) may be involved.

**Given the uncertainty, and because Theory B cannot be ruled out, it may be worth checking Power Platform/Microsoft 365 service health for this environment/tenant around today's date before or during the next session**, and considering whether this needs escalating to IT/your Power Platform admin if the same pattern recurs after a clean apply-and-publish.

## Diagnosis method

Rather than trying to reason about which values *should* be correct from memory, we cross-referenced every questionable action against a **31 July 2026 screenshot archive** (`Flow_B_Connectors.zip`, ~90 screenshots, Code view of each connector, captured systematically top-to-bottom during an earlier session) — this predates both today's build and the corruption, so it's a reliable "known good" baseline for anything that existed before today. For actions built fresh today (OF01–OF10, OF09-Gate and children), the baseline was instead what Claude and David had jointly built and verified via Code view earlier in today's own session.

We then did a **full top-to-bottom pass through every single connector in the flow**, cross-checked against "Go to operation"'s complete flat index to confirm nothing was missed, pasting each action's live Code view as plain text (far more reliable than screenshots — screenshots required OCR and were error-prone in a couple of cases) and classifying each as clean or needing a fix.

---

## Complete fix list (26 items, canvas order)

### Top-level initializers (before Condition_IsRecurring)

**1. `varFinalExistingPageSelfUrl`** (InitializeVariable)
```
Value: first(body('Filter_Existing_Mapping'))?['PageSelfUrl']
```

**2. `varFinalPageDecision`** (InitializeVariable)
```
Value: if(not(empty(outputs('Compose_PageSelfUrl_Created'))), 'PAGE_EXISTS', 'PAGE_NOT_FOUND')
```

*(Note: `varFinalMatchCount`, `varOutStatus`, `varOutputPageLink`, `varOutputPageSelfUrl`, `varTargetSectionPagesUrl`, `varOneNoteResolverResult`, `varPageAction` — all confirmed correctly blank-by-design at this top level; no fix needed for any of these.)*

### Condition_IsRecurring → True (recurring) branch

**3. `varFinalExistingPageSelfUrl_1`** (SetVariable)
```
Value: outputs('Compose_ExistingPageSelfUrl')
```

**4. `varFinalPageDecision_1`** (SetVariable)
```
Value: outputs('Compose_PageDecision')
```

**5. `varFinalMatchCount_1`** (SetVariable)
```
Value: string(outputs('Compose_Match_Count'))
```

### Condition_IsRecurring → False (one-off) branch

**6. `Set_varTargetSectionPagesUrl_OneOff_Exists`** (inside Condition_Section_Exists_OneOff → True → For_each_1)
```
Value: items('For_each_1')?['pagesUrl']
```

**7. `Set_varOneNoteResolverResult_Exists_OneOff`**
```
Value: ExistingSection    (plain text, exact case)
```

**8. `Set_varTargetSectionPagesUrl_OneOff_Created`** (inside Condition_Section_Exists_OneOff → False)
```
Value: outputs('Create_Section_OneOff')?['body']?['pagesUrl']
```

**9. `Set_varOneNoteResolverResult_Created_OneOff`**
```
Value: CreatedSection    (plain text, exact case)
```

**10. `OF03_—_Compose_PageDecision_OneOff`** — dash mismatch, not blank
```
Currently: outputs('OF02_-_Compose_ExistingPageSelfUrl_OneOff')   [hyphen — wrong]
Fix to:    outputs('OF02_—_Compose_ExistingPageSelfUrl_OneOff')   [em dash — correct]
```
Use the dynamic content/expression picker to insert this reference rather than retyping — guarantees the correct character.

**11. `OF04_—_Compose_Match_Count_OneOff`** — dash mismatch
```
Currently: body('OF01_-_Filter_Existing_Mapping_OneOff')   [hyphen]
Fix to:    body('OF01_—_Filter_Existing_Mapping_OneOff')   [em dash]
```

**12. `OF05a_—_Set_varFinalExistingPageSelfUrl_(OneOff)`**
```
Value: outputs('OF02_—_Compose_ExistingPageSelfUrl_OneOff')   [em dash]
```

**13. `OF05b_—_Set_varFinalPageDecision_(OneOff)`**
```
Value: outputs('OF03_—_Compose_PageDecision_OneOff')   [em dash]
```

**14. `OF05c_—_Set_varFinalMatchCount_(OneOff)`**
```
Value: outputs('OF04_—_Compose_Match_Count_OneOff')   [em dash]
```

### Condition_Mapping_Exists → True branch

**15. `Set_varTargetSectionPagesUrl_ExistingMapping`** — wrong field reference (Claude's own error, introduced in the OF10 fix earlier today)
```
Currently: triggerBody()?['IsRecurring']   [wrong — this is the field's title, not its key]
Fix to:    triggerBody()?['text']          [correct — matches trigger schema]
```
Full corrected expression:
```
if(equals(toLower(string(triggerBody()?['text'])), 'true'), first(body('Filter_Existing_Mapping'))?['SectionPagesUrl'], variables('varTargetSectionPagesUrl'))
```

*(Everything else in Condition_Mapping_Exists confirmed clean — Send_an_HTTP_request_to_SharePoint, Condition_Should_Write_Mapping, Condition_Section_Exists_Recurring and all its children, all Compose_* actions.)*

### Condition_Should_Create_Page → True branch

**16. `OF09-Gate_—_Condition_Is_Recurring_(SP_Write)`** — condition comparison reverted
```
Currently right-hand value: ""   [blank — wrong]
Fix to:                     true
```
Left-hand expression (`equals(toLower(string(triggerBody()?['text'])), 'true')`) is correct and unchanged.

**17. `OF09b-i_—_Condition_Should_Insert_Mapping_(OneOff)`** — dash mismatch in its own condition expression
```
Currently: body('OF01_-_Filter_Existing_Mapping_OneOff')   [hyphen]
Fix to:    body('OF01_—_Filter_Existing_Mapping_OneOff')   [em dash]
```

**18. `OF09b_—_HTTP_Update_SP_PageSelfUrl_(OneOff)`** — dash mismatch, two references in one URI expression
```
Currently: body('OF01_-_Filter_Existing_Mapping_OneOff')                     [hyphen]
Fix to:    body('OF01_—_Filter_Existing_Mapping_OneOff')                     [em dash]

Currently: body('OF09a_-_Send_an_HTTP_request_to_SharePoint_(OneOff)')       [hyphen]
Fix to:    body('OF09a_—_Send_an_HTTP_request_to_SharePoint_(OneOff)')       [em dash]
```

*(Create_OneNote_Page, Compose_PageSelfUrl_Created, HTTP_Update_SP_PageSelfUrl, Set_varPageAction_Created, Set_varOutputPageSelfUrl_Created, Set_varOutputPageLink_Created, OF09a — all confirmed clean.)*

### Condition_Should_Create_Page → False branch

**19. `Set_varPageAction_ExistsNoCreate`**
```
Value: Updated    (plain text, exact case)
```

**20. `Set_varOutputPageSelfUrl_Existing`** (this is OF08 — reverted, needs re-applying)
```
Value: variables('varFinalExistingPageSelfUrl')
```

**21. `Set_varPageAction_UpdatedAppend`**
```
Value: Updated    (plain text, exact case)
```

**22. `Set_varOutputPageLink_Existing`** (this is OF07 — reverted, needs re-applying)
```
Value: variables('varFinalExistingPageSelfUrl')
```

**23. `Set_varOutputPageLink_Created_OneOff`**
```
Value: outputs('Create_Page_OneOff')?['body']?['links']?['oneNoteWebUrl']?['href']
```

*(Compose_UpdateHtmlFragment, Compose_ExistingPageId, Condition_Is_Genuine_Existing_Page, Get_Sections_Existing_Branch, Filter_Existing_Section_By_Name, Apply_to_each_Existing_Section/Update_page_content_Existing_Branch, Create_Page_OneOff — all confirmed clean.)*

### Tail of flow

**24. `Set_varOutStatus`** (the second/later one, `runAfter: Compose_SP_Item_Count`)
```
Value: OK    (plain text)
```
Note: this restores the known pre-existing hardcoded placeholder. The full six-value OutStatus differentiation remains separate, already-tracked future work (per prior handover notes) — not something to build as part of this fix pass.

### Respond to the agent (final action)

These two are not corruption — they are pre-existing logic bugs, same root-cause family as OF06/OF07/OF08 (recurring-only `Compose_*` outputs referenced where the shared `varFinal*` variable should be used instead), just never caught until this full sweep.

**25. `outbranchresult`**
```
Currently: outputs('Compose_Match_Count')          [recurring-only, undefined for one-off]
Fix to:    variables('varFinalMatchCount')
```

**26. `outpageroute`**
```
Currently: equals(outputs('Compose_PageDecision'), 'PAGE_EXISTS')       [recurring-only]
Fix to:    equals(variables('varFinalPageDecision'), 'PAGE_EXISTS')
```

*(All other Respond fields — outisrecurring, outmeetingtitle, outseriesmasterid, outpagehtml, outspitemcount, outonenoteresolverresult, outtargetsectionpagesurl, outcreatedpagelink, outcreatedpageselfurl, outfinaltargetsectionpagesurl, outresolverresult, outexistingpageselfurl, outpagedecision, outpageaction, outupdatehtmlfragment, outagentresponsesummary, outstatus — all confirmed clean.)*

---

## Also noted, not fixed (separate, lower-priority defect — logged for future reference)

**`Compose_AgentResponseSummary`** has a real value and is not corrupted, but its `if()` logic checks `variables('varPageAction')` against `'UpdatedAppend'` and `'ExistsNoCreate'` — values that are never actually set anywhere in the flow (the real values set are `'Updated'` in both cases, per fixes #19/#21 above). This means the summary text always falls through to the generic "Processed OneNote meeting page request..." message rather than the more specific wording. Cosmetic only — does not affect functionality — but worth fixing in a future session for message-quality reasons.

---

## Status

**Fix list complete, cross-verified against 31 July archive and today's own build record. Not yet applied.** Next session: apply all 26, verify exhaustively (Flow Checker + individual re-check), then publish and run the three test scenarios from the prior handover doc.
