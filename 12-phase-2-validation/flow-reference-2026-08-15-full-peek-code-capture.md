# Flow reference — complete structure + Peek Code capture (15 August 2026, session 2)

Purpose: this is the most complete snapshot of "PA - Resolve OneNote Meeting Section - v2 Clean Build" captured to date — full structural map plus Peek Code for every action in the flow, not just the 26 previously identified at-risk actions. Use this as the reference point for rebuilding/verifying the flow if corruption strikes again, instead of relying on Version History alone.

**Captured from**: the draft state at time of capture — note this was **after** Incident 4's corruption had struck, so all `SetVariable` actions below show their *corrupted* `""` values as captured. The **correct expression** column gives the intended value, reconstructed from (a) the pre-corruption capture earlier this session where available, or (b) direct structural inference from parallel/mirrored branches where not. Entries marked **INFERRED** were not in the original 26-item list and should be double-checked in Designer before writing back.

---

## Flow structure (top-level)

```
Trigger (When an agent calls the flow)
  └─ Get items (SharePoint, list RecurringMeetingSectionMap)
     └─ InitializeVariable x10:
         varFinalExistingPageSelfUrl, varFinalPageDecision, varFinalMatchCount,
         varOutStatus (= "OK" at init - see note below), varOutputPageLink,
         varOutputPageSelfUrl, varTargetSectionPagesUrl, varOneNoteResolverResult,
         varPageAction
         └─ Condition Mapping Exists (expr: length(body('Get_items'))>0 ? true/false via IsRecurring gate)
              ├─ TRUE (recurring path):
              │   Compose_Input_SeriesMasterId → Compose_Input_MeetingTitle → Filter_Existing_Mapping → Compose_ExistingPageSelfUrl → Compose_PageDecision → Compose_Match_Count → varFinalExistingPageSelfUrl_1 / varFinalPageDecision_1 / varFinalMatchCount_1
              └─ FALSE (one-off path):
                  FB-F01 Compose MeetingTitle (one-off) → Get_Sections_OneOff → Filter_OneNote_Section_OneOff → Compose_Section_Match_Count_OneOff → Condition_Section_Exists_OneOff
                      ├─ TRUE: For_each_1 → Set_varTargetSectionPagesUrl_OneOff_Exists / Set_varOneNoteResolverResult_Exists_OneOff
                      └─ FALSE: Create_Section_OneOff → Set_varTargetSectionPagesUrl_OneOff_Created / Set_varOneNoteResolverResult_Created_OneOff
                  → OF01 Filter_Existing_Mapping_OneOff → OF02-OF04 Compose chain → OF05a/b/c SetVariable chain
     └─ Condition IsRecurring (gates Compose_Branch_Result vs _NoMatch, and Condition_Recurring_TargetSection / Condition_Should_Write_Mapping)
         └─ Condition_Section_Exists_Recurring → Create_Section_Recurring or Apply_to_each (mirrors OneOff logic)
   └─ Condition Should Create Page (expr: varFinalPageDecision == 'PAGE_NOT_FOUND')
       ├─ TRUE: Create_OneNote_Page → Compose_PageSelfUrl_Created → OF09-Gate (Condition Is Recurring - SP Write)
       └─ FALSE: Set_varPageAction_ExistsNoCreate → Set_varOutputPageSelfUrl_Existing → Compose_UpdateHtmlFragment → Compose_ExistingPageId → Condition_Is_Genuine_Existing_Page
           ├─ TRUE: Update existing page (Get_Sections_Existing_Branch chain)
           └─ FALSE: Create_Page_OneOff (this is where Bug 5 surfaces — sectionId = variables('varTargetSectionPagesUrl'))
   └─ Compose_AgentResponseSummary → Compose_SP_Item_Count → Set_varOutStatus (= "OK") → Respond to the agent
```

---

## Full expression reference table

> **Column key:** `Captured value` = what Peek Code showed this session (post-corruption). `Correct expression` = what should be written back. `Source` = how we know.

### InitializeVariable actions (no value field by design, except varOutStatus)

| Action | Captured value | Note |
|---|---|---|
| `varFinalExistingPageSelfUrl` | (no value field) | Normal |
| `varFinalPageDecision` | (no value field) | Normal |
| `varFinalMatchCount` | (no value field) | Normal |
| `varOutStatus` | `"OK"` | Set at initialization — separate from the downstream `Set_varOutStatus` SetVariable action which also sets `"OK"` later in the flow. Both currently read OK; unconfirmed functionally this session. |
| `varOutputPageLink` | (no value field) | Normal |
| `varOutputPageSelfUrl` | (no value field) | Normal |
| `varTargetSectionPagesUrl` | (no value field) | Normal — this is one of the actions disproportionately affected by corruption incidents |
| `varOneNoteResolverResult` | (no value field) | Normal — same as above |
| `varPageAction` | (no value field) | Normal |

### SetVariable actions — recurring/mapping-exists branch

| Action | Captured value | Correct expression | Source |
|---|---|---|---|
| `varFinalExistingPageSelfUrl_1` | `""` | `@outputs('Compose_ExistingPageSelfUrl')` | **INFERRED** — mirrors OF05a pattern |
| `varFinalPageDecision_1` | `""` | `@outputs('Compose_PageDecision')` | **INFERRED** — mirrors OF05b pattern |
| `varFinalMatchCount_1` | `""` | `@string(outputs('Compose_Match_Count'))` | **INFERRED** — mirrors OF05c pattern |
| `Set_varOneNoteResolverResult_ExistingMapping` | `"ExistingMapping"` | `ExistingMapping` (literal, already correct) | Not corrupted — literal value survived |
| `Set_varTargetSectionPagesUrl_ExistingMapping` | `@first(body('Filter_Existing_Mapping'))?['SectionPagesUrl']` | (already correct) | Not corrupted — expression survived |
| `varTargetSectionPagesUrl_1` | `""` | `@items('Apply_to_each')?['pagesUrl']` | **INFERRED** — mirrors OneOff `Set_varTargetSectionPagesUrl_OneOff_Exists` pattern |
| `varOneNoteResolverResult_1` | `""` | `ExistingSection` | **INFERRED** — mirrors OneOff equivalent |
| `varTargetSectionPagesUrl_2` | `""` | `@outputs('Create_Section_Recurring')?['body']?['pagesUrl']` | **INFERRED** — mirrors OneOff `_Created` pattern |
| `varOneNoteResolverResult_2` | `""` | `CreatedSection` | **INFERRED** — mirrors OneOff equivalent |

### SetVariable actions — one-off branch (confirmed from pre-corruption capture earlier this session)

| Action | Captured value | Correct expression |
|---|---|---|
| `Set_varTargetSectionPagesUrl_OneOff_Exists` | `""` | `@items('For_each_1')?['pagesUrl']` |
| `Set_varOneNoteResolverResult_Exists_OneOff` | `""` | `ExistingSection` |
| `Set_varTargetSectionPagesUrl_OneOff_Created` | `""` | `@outputs('Create_Section_OneOff')?['body']?['pagesUrl']` |
| `Set_varOneNoteResolverResult_Created_OneOff` | `""` | `CreatedSection` |
| `OF05a — Set varFinalExistingPageSelfUrl (OneOff)` | `""` | `@outputs('OF02_—_Compose_ExistingPageSelfUrl_OneOff')` |
| `OF05b — Set varFinalPageDecision (OneOff)` | `""` | `@outputs('OF03_—_Compose_PageDecision_OneOff')` |
| `OF05c — Set varFinalMatchCount (OneOff)` | `""` | `@string(outputs('OF04_—_Compose_Match_Count_OneOff'))` |

### SetVariable actions — page creation / update branch (confirmed from pre-corruption capture)

| Action | Captured value | Correct expression |
|---|---|---|
| `Set_varPageAction_Created` | `""` | `Created` |
| `Set_varOutputPageSelfUrl_Created` | `""` | `@outputs('Compose_PageSelfUrl_Created')` |
| `Set_varOutputPageLink_Created` | `""` | `@outputs('Create_OneNote_Page')?['body']?['links']?['oneNoteWebUrl']?['href']` |
| `Set_varPageAction_Created_OneOff` | `""` | `Created` |
| `Set_varOutputPageSelfUrl_Created_OneOff` | `""` | `@outputs('Compose_PageSelfUrl_Created')` |
| `Set_varOutputPageLink_Created_OneOff_Gate` | `""` | `@outputs('Create_OneNote_Page')?['body']?['links']?['oneNoteWebUrl']?['href']` |
| `Set_varPageAction_ExistsNoCreate` | `""` | `Updated` |
| `Set_varOutputPageSelfUrl_Existing` | `""` | `@variables('varFinalExistingPageSelfUrl')` |
| `Set_varPageAction_UpdatedAppend` | `""` | `Updated` |
| `Set_varOutputPageLink_Existing` | `""` | `@variables('varFinalExistingPageSelfUrl')` |
| `Set_varOutputPageLink_Created_OneOff` | `""` | `@outputs('Create_Page_OneOff')?['body']?['links']?['oneNoteWebUrl']?['href']` |
| `Set_varOutStatus` (final, before Respond to the agent) | `"OK"` | `OK` (already correct — see Bug 8 note below) |

---

## Key connectors and expressions (not variable-corruption-affected, for reference)

- **Get items**: SharePoint GetItems, dataset `https://jsainsbury.sharepoint.com/sites/coplt`, table `186b3c9f-e758-4e85-83d5-685946614a0a`
- **Create_OneNote_Page**: OneNote CreatePageInSection, `sectionId: @variables('varTargetSectionPagesUrl')`, notebookKey `Meeting Notes|$|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Meeting Notes`
- **Create_Page_OneOff**: same connector/notebook, same `sectionId` source — this is the Bug 5 failure point when `varTargetSectionPagesUrl` is blank
- **Respond to the agent (Response action)**: full output schema includes `outstatus: @{variables('varOutStatus')}` plus 18 other output fields (outisrecurring, outmeetingtitle, outseriesmasterid, outpagehtml, outspitemcount, outmatchcount, outbranchresult, outonenoteresolverresult, outtargetsectionpagesurl, outcreatedpagelink, outcreatedpageselfurl, outfinaltargetsectionpagesurl, outresolverresult, outexistingpageselfurl, outpagedecision, outpageroute, outpageaction, outupdatehtmlfragment, outagentresponsesummary)

---

## Bug 8 note

There are **two** places `varOutStatus` gets a value: the `InitializeVariable` declaration (value `OK`, top of flow) and the `SetVariable` "Set varOutStatus" (also value `OK`, near the end, right before Respond to the agent). Both currently read `OK`. This is consistent, not contradictory — but functional confirmation via a test run that actually reaches the final `Set_varOutStatus` action is still outstanding as of end of session 2.

## Bug 5 root cause (confirmed this session)

`Create_Page_OneOff`'s `sectionId` parameter is `@variables('varTargetSectionPagesUrl')`. That variable is set exclusively by the corrupted SetVariable actions in this document. When those are corrupted to `""`, `Create_Page_OneOff` inherits the blank value and fails exactly as observed in the run history ("The section id given in the input is invalid"). Bug 5 and the corruption pattern are the same root cause, not two separate bugs.

---

*Captured 15 August 2026, session 2. Cross-reference with `handover-2026-08-15-session2-corruption-mechanism-identified.md` and `MICROSOFT-SUPPORT-TICKET-DRAFT-2026-08-15.md`.*
