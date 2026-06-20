# Living Audit — Per-Action Expression Catalogue

See `PROCESS-expression-audit-maintenance.md` for the maintenance rules governing this document. Short version: this is current ground truth, not a session log. Update it the moment an expression changes in Designer, before closing out the session's handover note.

**Last updated:** 2026-06-20 (evening session)
**Coverage:** Flow B (`PA - Resolve OneNote Meeting Section - v2 Clean Build`, flowId `ed112c88-b94b-f111-bec6-002248a38052`) — primary path complete. Flow A — only the four actions fixed/touched on 2026-06-20 are logged here; a full systematic pass of Flow A has not yet been done.

**Status key:** 🔴 confirmed bug, not fixed · 🟡 suspect/unconfirmed · 🟢 confirmed fixed and tested · ⚪ confirmed clean (no issue)

---

## Flow B — Trigger

### `When an agent calls the flow` 🔴
Generic field names: `text`, `text_1`, `text_2`, `text_3`, `text_4` with display titles MeetingTitle / SeriesMaster / PageHtml / MeetingId / IsRecurring respectively (exact text_N→title mapping not yet 100% confirmed — verify via C8B/C10 input panel before editing).
**Bug:** `Condition_IsRecurring` reads `triggerBody()?['IsRecurring']`, which does not exist as a key — the real key is `text` (or whichever text_N maps to the IsRecurring title). Confirmed via `Respond_to_agent`'s `outisrecurring: "@{triggerBody()?['text']}"` mapping.
**Fix (not yet applied):** Change `Condition_IsRecurring`'s expression to read the correct generic key once confirmed.

---

## Flow B — Condition_IsRecurring

### `Condition_IsRecurring` (the condition itself) 🔴
```
toLower(string(triggerBody()?['IsRecurring'])) is equal to "true"
```
Same bug as above — `triggerBody()?['IsRecurring']` always null. See trigger entry.

### True branch

**`Compose Input SeriesMasterId`** ⚪ — clean, not yet transcribed verbatim.

**`Compose Input MeetingTitle`** ⚪ — clean, not yet transcribed verbatim.

**`Filter Existing Mapping`** ⚪
- From: `value` (Get_items output)
- Filter Query: `SeriesMasterId` is equal to `SeriesMasterId` (dynamic content, from Compose Input SeriesMasterId)

**`Compose ExistingPageSelfUrl`** ⚪ — `if(...)` expression, content not yet transcribed verbatim.

**`Compose PageDecision`** ⚪ — `if(...)` expression, content not yet transcribed verbatim.

**`Compose Match Count`** ⚪ — `length(...)` expression, content not yet transcribed verbatim.

**`varFinalExistingPageSelfUrl_1`** 🔴 — blank Value, `'Value' is required`.

**`varFinalPageDecision_1`** 🔴 — blank Value, `'Value' is required`.

**`varFinalMatchCount_1`** 🔴 — blank Value (`"value": ""`), `'Value' is required`. **This is the root cause feeding the `Condition_Should_Write_Mapping` crash** — see Condition Mapping Exists section below.

### False branch

**`FB-F01 — Compose Input MeetingTitle (one-off)`** ⚪ — `concat(...)` expression, content not yet transcribed verbatim.

**`Get Sections OneOff`** ⚪ — Notebook Key: Meeting Notes.

**`Filter OneNote Section OneOff`** ⚪
- From: `body/value`
- Filter Query: `name` is equal to `Outputs` (dynamic)

**`Compose Section Match Count OneOff`** ⚪ — `length(...)` expression.

**`Condition Section Exists OneOff`** 🔴 (condition itself)
```
greater(...) is equal to true
```
Same bug family as `Condition_Should_Write_Mapping` — needs the expanded `greater(...)` content transcribed and checked for the same missing-`empty()`-guard pattern.

True branch (section exists):
- **`For each 1`** ⚪ — iterates `body/value`.
- **`Set varTargetSectionPagesUrl OneOff Exists`** 🔴 — blank Value.
- **`Set varOneNoteResolverResult Exists OneOff`** 🔴 — blank Value.

False branch (section doesn't exist):
- **`Create Section OneOff`** ⚪ — Notebook Key: Meeting Notes, Name: `Outputs` (dynamic).
- **`Set varTargetSectionPagesUrl OneOff Created`** 🔴 — blank Value.
- **`Set varOneNoteResolverResult Created OneOff`** 🔴 — blank Value.

---

## Flow B — Condition Mapping Exists

### `Condition Mapping Exists` (the condition itself) ⚪ — confirmed working pattern
```
if(empty(coalesce(variables('varFinalMatchCount'), '')), '0', greater(int(coalesce(variables('varFinalMatchCount'), '0')), 0)) is equal to true
```
(Reconstructed from description — exact text not yet pasted into this doc verbatim; functionally confirmed correct via comparison against the broken sibling below, since this one has the `empty()` guard and the sibling doesn't.)

### True branch (mapping exists)
- **`Compose Branch Result`** ⚪
- **`Set varTargetSectionPagesUrl ExistingMapping`** ⚪ — has Value populated (not blank).
- **`Set varOneNoteResolverResult ExistingMapping`** ⚪ — has Value populated (not blank).

### False branch (no mapping)
- **`Compose Branch Result NoMatch`** ⚪

- **`Condition_Should_Write_Mapping`** 🔴 **CONFIRMED ROOT CAUSE OF LIVE CRASH (2026-06-20)**
```json
{
  "type": "If",
  "expression": {
    "and": [
      {
        "equals": [
          "@greater(\n  int(\n    coalesce(\n      variables('varFinalMatchCount'),\n      '0'\n    )\n  ),\n  0\n)",
          "@true"
        ]
      }
    ]
  }
}
```
**Bug:** `coalesce(variables('varFinalMatchCount'), '0')` only substitutes when the variable is **null**. `varFinalMatchCount_1` is initialized with `"value": ""` (empty string, not null) — see above. `coalesce` passes the empty string straight through, so `int('')` throws `InvalidTemplate` at runtime. This is the confirmed cause of tonight's manual-test crash.

**Fix (drafted, not yet applied):**
```
greater(int(if(empty(variables('varFinalMatchCount')), '0', variables('varFinalMatchCount'))), 0)
```
Matches the working pattern used by `Condition Mapping Exists` above.

  True branch:
  - **`Send_an_HTTP_request_to_SharePoint`** ⚪ — confirmed clean once the guard above is fixed. Full body:
    ```json
    {
      "Title": "Mapping",
      "SeriesMasterId": "@{outputs('Compose_Input_SeriesMasterId')}",
      "MeetingTitle": "@{outputs('Compose_Input_MeetingTitle')}",
      "SectionPagesUrl": "@{variables('varTargetSectionPagesUrl')}",
      "Status": "Active"
    }
    ```
    POST to `_api/web/lists/GetByTitle('RecurringMeetingSectionMap')/items`. `runAfter: Compose_SafeSectionName: ["SUCCEEDED"]` — note casing bug, should be `"Succeeded"` (see Flow B-wide issues below).

  False branch: empty (`"actions": {}`) — no action needed, by design.

- **`Compose IgnoreSeriesMasterId`** 🔴 — `"inputs": "''"` (literal 2-character string, same bug family as historic FA33A-class bugs). Fix: should be a real expression or `string('')`, never confirmed what the intended value is — needs design decision, not just a syntax fix.
- **`Compose PageRoute CreateRequired`** ⚪ — not yet transcribed verbatim.
- **`Compose SectionDisplayName`** ⚪ — not yet transcribed verbatim.
- **`Compose SafeSectionName`** ⚪ — not yet transcribed verbatim.

---

## Flow B — True-branch page-creation chain (Condition Should Create Page)

### `Condition Should Create Page` (the condition itself) 🟡 unconfirmed, suspect
```
equals(...) is equal to true
```
Structurally matches the same pattern family as the other two conditions above. The contents of the `equals(...)` chip have not yet been expanded/transcribed — needs checking for the same missing-guard issue before ruling it clean.

### True branch (create page)
- **`Create OneNote Page`** ⚪
- **`Compose PageSelfUrl Created`** ⚪ — Inputs: `self` (dynamic).
- **`HTTP Update SP PageSelfUrl`** ⚪ — POST to `_api/web/lists/GetByTitle('RecurringMeetingSectionMap')/items(...)`, headers include `IF-MATCH: *`, `X-HTTP-Method: MERGE`. Site: `Product SCLIF - https://jsainsbury.sharepoint.com/sites/coplt`.
- **`Set varPageAction Created`** 🔴 — blank Value.
- **`Set varOutputPageSelfUrl Created`** 🔴 — blank Value.
- **`Compose UpdateHtmlFragment`** ⚪ — static HTML fragment with "Automated update" / "Meeting Capture Agent" text, no dynamic bug risk.
- **`Compose ExistingPageId`** ⚪ — not yet transcribed verbatim.
- **`Set varOutputPageLink Created`** 🔴 — blank Value. **This is THIS MORNING'S CONFIRMED FIX, found blank again tonight (2026-06-20) — genuine regression, not stale capture.** Intended expression (re-apply): `outputs('Create_Page_OneOff')?['body']?['links']?['oneNoteWebUrl']?['href']` — note: verify this is the right output reference for `Create_OneNote_Page` (non-OneOff) vs `Create_Page_OneOff`, as this chain uses the former.

### Condition Is Genuine Existing Page (nested inside False branch of Should Create Page, i.e. page-exists path)

**`Condition Is Genuine Existing Page`** ⚪ — condition expression confirmed clean:
```
equals(...) is equal to true
```
True/False branches both present.

True branch:
- **`Get Sections Existing Branch`** ⚪
- **`Create Page OneOff`** ⚪ — Notebook Key: Meeting Notes, Notebook section: `varTargetSectio...` (dynamic), Page Content: `PageHtml` (dynamic).
- **`Set varOutputPageLink Created OneOff`** 🔴 — blank Value. Paired with the `Create Page OneOff` fix from earlier today (the one-off page-creation root cause). Intended expression: `outputs('Create_Page_OneOff')?['body']?['links']?['oneNoteWebUrl']?['href']`.

False branch / existing-section update path:
- **`Filter Existing Section By Name`** ⚪ — From: `body/value`, Filter Query: `name` equal to `Outputs`.
- **`Apply to each Existing Section`** ⚪ — iterates `Body`.
  - **`Update page content Existing Branch`** ⚪ **confirmed clean (2026-06-20)** — Notebook Key: Meeting Notes, Notebook section: `id` (dynamic), Page Id: `Outputs` (dynamic). Update: Target `body`, Action `append`, Location `after`, Content `Outputs` (dynamic). No blank-value issues.
- **`Set varPageAction UpdatedAppend`** 🔴 — blank Value.
- **`Set varOutputPageLink Existing`** 🔴 — blank Value.

(Also present per earlier audit, not yet individually re-confirmed this pass: `Set varOutputPageSelfUrl Existing`, `Compose UpdateHtmlFragment` [existing-page variant], `Compose ExistingPageId`.)

---

## Flow B — Final response

### `Compose AgentResponseSummary` ⚪
`if(...)` chain checking `variables('varPageAction')` against `'Created'` / `'UpdatedAppend'` / `'ExistsNoCreate'`. **Logic itself is clean**, but currently always falls through to the else/generic message because `varPageAction` is never actually set (all the `Set varPageAction *` actions above are blank). Will self-resolve once those are fixed — no separate fix needed here.

### `Compose SP Item Count` ⚪ — `length(...)` expression.

### `Respond to the agent` ⚪ — full output schema confirmed clean:
| Output | Source |
|---|---|
| OutIsRecurring | `IsRecurring` (trigger field, dynamic) |
| OutMeetingTitle | `MeetingTitle` (trigger field, dynamic) |
| OutSeriesMasterId | `SeriesMasterId` (trigger field, dynamic) |
| OutPageHtml | `PageHtml` (trigger field, dynamic) |
| OutSPItemCount | `int(...)` expression |
| OutMatchCount | `varFinalMatchCo...` (dynamic, truncated in UI) |
| (additional outputs below the fold not yet re-confirmed this pass) | |

Good reference pattern: this action's match-count-style outputs use `coalesce(outputs(...), 0)` with a numeric fallback — cleaner than `Condition_Should_Write_Mapping`'s broken version. Useful as the template when fixing the blank-Value Set-variable actions above.

---

## Flow B-wide issues (not tied to one action)

🔴 **`runAfter` casing bug** — `"SUCCEEDED"` used instead of `"Succeeded"`. Confirmed repeated extremely consistently: all 8 init-variable actions, `Get_Sections_OneOff`, `Filter_OneNote_Section_OneOff`, `Compose_Section_Match_Count_OneOff`, `Create_Section_OneOff` and its Set-variable children, `Condition_Should_Write_Mapping`'s own runAfter, `Send_an_HTTP_request_to_SharePoint`'s runAfter (confirmed 2026-06-20 evening). Needs a flow-wide find/replace pass once other fixes are stable — low risk individually but high count.

🟡 **Naming/type concern, unconfirmed** — `Create_OneNote_Page` and `Create_Page_OneOff` both pass `sectionId: "@variables('varTargetSectionPagesUrl')"`. Variable name suggests URL, field name suggests it should be a section ID. Not confirmed broken, just flagged as suspicious, especially since the variable is also sometimes left blank.

⚪ **Dead conditional logic, low priority** — `Set_varTargetSectionPagesUrl_ExistingMapping`'s if/else branches are identical. Not a bug, just redundant; can be simplified whenever that action is next touched for another reason.

---

## Flow A — actions touched 2026-06-20 (not a full Flow A pass)

### `FA28A_Compose_OutIsRecurring` 🟢 fixed, confirmed live
- Was: `coalesce(outputs('FA28_Compose_SingleEvent')?['IsRecurring'], 'false')` — read a nonexistent field.
- Now: `if(empty(coalesce(outputs('FA28_Compose_SingleEvent')?['seriesMasterId'], '')), 'false', 'true')`
- Verified: live Teams test, "QWE Meeting" (confirmed recurring series), returned `isrecurring: "true"`.

### `FA28B_Compose_OutSeriesMasterId` 🟢 fixed, confirmed live
- Was: `?['SeriesMasterId']` (wrong case).
- Now: `?['seriesMasterId']`.
- Verified: same live test as above, `seriesmasterid` populated correctly.

### `FA33A_Set_varCandidateListText_Empty` 🟢 fixed
- Was: blank/literal `''`.
- Now: `string('')`.

### `FA34A_Set_varCandidateIndex_One` 🟢 fixed
- Was: blank.
- Now: `1`.

**Outstanding for Flow A:** a full systematic Code View pass (matching the depth done for Flow B above) has not yet happened. Given the same three bug patterns were found repeatedly in Flow B, Flow A should be assumed to have similar undiscovered issues until proven otherwise.

---

## Open items / not yet covered by this audit

- Trigger field → Topic variable mapping for `text`/`text_1`–`text_4` (needed before the `Condition_IsRecurring` fix can be safely applied).
- Several `⚪ clean, not yet transcribed verbatim` entries above should be filled in with exact expression text next time they're opened, for completeness.
- Full Flow A systematic pass.
- Live re-test of all fixes together via a brand-new Teams thread (not the connection-incident-contaminated one).
