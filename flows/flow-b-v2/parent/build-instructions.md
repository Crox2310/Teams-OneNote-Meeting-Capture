# Flow B v2 — Parent Build Instructions

**Flow name:** `PA - Meeting Capture - B - Router - v2`
**Build methodology:** Compose-based state passing. No SetVariable. No variables.
**Session-start check:** open flow, wait 20s, Flow Checker, Peek Code `Scope_FlowB_Parent`.

---

## Prerequisites before opening Power Automate

1. Read `flows/flow-b-v2/parent/trigger-contract.md` — you need the 8 input fields and their titles.
2. Read `flows/flow-b-v2/parent/output-contract.md` — you need the 20 output fields and their exact keys.
3. Confirm both child flows are created (PowerAppV2 trigger, solution-aware, Respond action present, connectors set to "Use this connection" in Run-only-users). The parent's Run-a-Child-Flow actions will not be available in the picker until this is done.

---

## Step 1 — Create the flow

1. In Power Automate, create a new flow.
2. Trigger: **Copilot Studio** — "When Copilot Studio calls a flow" (the Skills trigger — same kind as v1).
3. Name: `PA - Meeting Capture - B - Router - v2`.
4. Add the 8 input fields **in order** (platform assigns keys sequentially — order is critical):

| Title | Type | Required |
|---|---|---|
| IsRecurring | Text | Yes |
| MeetingTitle | Text | Yes |
| SeriesMasterId | Text | Yes |
| PageHtml | Text | Yes |
| MeetingId | Text | Yes |
| OccurrenceDate | Text | No |
| EndTime | Text | No |
| JoinUrl | Text | No |

5. Peek Code the trigger immediately. Confirm keys: `text`=IsRecurring, `text_1`=MeetingTitle, `text_2`=SeriesMasterId, `text_3`=PageHtml, `text_4`=MeetingId, `text_5`=OccurrenceDate, `text_6`=EndTime, `text_7`=JoinUrl. **If any key is wrong, delete and re-add inputs in the correct order before proceeding.**

---

## Step 2 — Outer Scope

1. Add a **Scope** action.
2. Name: `Scope_FlowB_Parent`
3. Description: `Full parent router — Peek Code this for a full-flow health check`
4. All subsequent actions go inside this Scope.

---

## Step 3 — Inner Scope: Router

1. Inside `Scope_FlowB_Parent`, add a **Scope**.
2. Name: `Scope_Router`
3. Description: `Derive route, branch, call one child`

### RT01 — Compose Route Key

4. Inside `Scope_Router`, add a **Compose**.
5. Name: `RT01 Compose Route Key`
6. Inputs (Expression tab):
```
if(empty(triggerBody()?['text_2']), 'ONEOFF', 'RECURRING')
```
7. Peek Code. Confirm `"inputs": "@if(empty(triggerBody()?['text_2']), 'ONEOFF', 'RECURRING')"`

### RT02 — Condition Route

8. Add a **Condition** after RT01, still inside `Scope_Router`.
9. Name: `RT02 Condition Route`
10. Configure using the standard two-field builder (not advanced mode — advanced mode wraps standalone booleans in `equals("")`):
 - Left: Expression tab: `outputs('RT01_Compose_Route_Key')`
 - Operator: `is equal to`
 - Right: Expression tab (or value): `ONEOFF`
11. Peek Code. Confirm:
```json
"expression": {
  "and": [{
    "equals": [
      "@outputs('RT01_Compose_Route_Key')",
      "ONEOFF"
    ]
  }]
}
```

### RT03b — Run OneOff Child (True branch)

12. In the **True** branch of RT02, add a **Run a Child Flow** action.
13. Name: `RT03b Run OneOff Child`
14. Child flow: select `PA - Meeting Capture - B - OneOff - v2` from the picker.
15. Map inputs (all Expression tab):

| Child input | Expression |
|---|---|
| MeetingTitle (`text`) | `triggerBody()?['text_1']` |
| MeetingId (`text_1`) | `triggerBody()?['text_4']` |
| OccurrenceDate (`text_2`) | `triggerBody()?['text_5']` |
| PageHtml (`text_3`) | `triggerBody()?['text_3']` |
| EndTime (`text_4`) | `triggerBody()?['text_6']` |
| JoinUrl (`text_5`) | `triggerBody()?['text_7']` |

16. Peek Code. Confirm all 6 input expressions are present and correct.

### RT03a — Run Recurring Child (False branch)

17. In the **False** branch of RT02, add a **Run a Child Flow** action.
18. Name: `RT03a Run Recurring Child`
19. Child flow: select `PA - Meeting Capture - B - Recurring - v2` from the picker.
20. Map inputs (all Expression tab):

| Child input | Expression |
|---|---|
| MeetingTitle (`text`) | `triggerBody()?['text_1']` |
| SeriesMasterId (`text_1`) | `triggerBody()?['text_2']` |
| OccurrenceDate (`text_2`) | `triggerBody()?['text_5']` |
| PageHtml (`text_3`) | `triggerBody()?['text_3']` |
| EndTime (`text_4`) | `triggerBody()?['text_6']` |
| JoinUrl (`text_5`) | `triggerBody()?['text_7']` |

21. Peek Code. Confirm all 6 input expressions correct.

---

## Step 4 — Inner Scope: Relay

1. After `Scope_Router` (but still inside `Scope_FlowB_Parent`), add a **Scope**.
2. Name: `Scope_Relay`
3. Description: `Assemble 20-field v1 output shape and respond`
4. Set runAfter on `Scope_Relay` to run after `Scope_Router` on **Succeeded AND Failed** (so a child failure still reaches the Respond).

### RL01–RL04 — Parent echo Composes

5. Add 4 **Compose** actions inside `Scope_Relay`:

| Ref | Name | Expression |
|---|---|---|
| RL01 | `RL01 Compose Out IsRecurring` | `triggerBody()?['text']` |
| RL02 | `RL02 Compose Out MeetingTitle` | `triggerBody()?['text_1']` |
| RL03 | `RL03 Compose Out SeriesMasterId` | `triggerBody()?['text_2']` |
| RL04 | `RL04 Compose Out PageHtml` | `triggerBody()?['text_3']` |

### RL05–RL20 — Child relay Composes

6. Add 16 **Compose** actions. For each, use the coalesce-across-both-children pattern with a `''` terminal fallback (except `outstatus` which uses `'ERROR'`).

Pattern (replace `<key>` with the field key):
```
coalesce(body('RT03a_Run_Recurring_Child')?['<key>'], body('RT03b_Run_OneOff_Child')?['<key>'], '')
```

| Ref | Name | Key | Fallback |
|---|---|---|---|
| RL05 | `RL05 Compose Out SP Item Count` | `outspitemcount` | `''` |
| RL06 | `RL06 Compose Out Match Count` | `outmatchcount` | `''` |
| RL07 | `RL07 Compose Out Branch Result` | `outbranchresult` | `''` |
| RL08 | `RL08 Compose Out OneNote Resolver Result` | `outonenoteresolverresult` | `''` |
| RL09 | `RL09 Compose Out Target Section Pages Url` | `outtargetsectionpagesurl` | `''` |
| RL10 | `RL10 Compose Out Created Page Link` | `outcreatedpagelink` | `''` |
| RL11 | `RL11 Compose Out Created Page Self Url` | `outcreatedpageselfurl` | `''` |
| RL12 | `RL12 Compose Out Final Target Section Pages Url` | `outfinaltargetsectionpagesurl` | `''` |
| RL13 | `RL13 Compose Out Resolver Result` | `outresolverresult` | `''` |
| RL14 | `RL14 Compose Out Existing Page Self Url` | `outexistingpageselfurl` | `''` |
| RL15 | `RL15 Compose Out Page Decision` | `outpagedecision` | `''` |
| RL16 | `RL16 Compose Out Page Route` | `outpageroute` | `''` |
| RL17 | `RL17 Compose Out Page Action` | `outpageaction` | `''` |
| RL18 | `RL18 Compose Out Update Html Fragment` | `outupdatehtmlfragment` | `''` |
| RL19 | `RL19 Compose Out Agent Response Summary` | `outagentresponsesummary` | `''` |
| RL20 | `RL20 Compose Out Status` | `outstatus` | `'ERROR'` |

7. Peek Code each. Confirm the coalesce expression is present and the correct key name appears.

### RL_Respond — Respond to the agent

8. Add a **"Respond to the agent"** (Skills Response) action — this is the Copilot Studio response action, same kind as v1. **Not** "Respond to a PowerApp or Flow."
9. Name: `RL_Respond`
10. Status code: `200`
11. Add all 20 output fields **in v1 key order**, binding each to its RL Compose output:

| Key | Title | Value (Expression tab) |
|---|---|---|
| `outisrecurring` | OutIsRecurring | `outputs('RL01_Compose_Out_IsRecurring')` |
| `outmeetingtitle` | OutMeetingTitle | `outputs('RL02_Compose_Out_MeetingTitle')` |
| `outseriesmasterid` | OutSeriesMasterId | `outputs('RL03_Compose_Out_SeriesMasterId')` |
| `outpagehtml` | OutPageHtml | `outputs('RL04_Compose_Out_PageHtml')` |
| `outspitemcount` | OutSPItemCount | `outputs('RL05_Compose_Out_SP_Item_Count')` |
| `outmatchcount` | OutMatchCount | `outputs('RL06_Compose_Out_Match_Count')` |
| `outbranchresult` | OutBranchResult | `outputs('RL07_Compose_Out_Branch_Result')` |
| `outonenoteresolverresult` | OutOneNoteResolverResult | `outputs('RL08_Compose_Out_OneNote_Resolver_Result')` |
| `outtargetsectionpagesurl` | OutTargetSectionPagesUrl | `outputs('RL09_Compose_Out_Target_Section_Pages_Url')` |
| `outcreatedpagelink` | OutCreatedPageLink | `outputs('RL10_Compose_Out_Created_Page_Link')` |
| `outcreatedpageselfurl` | OutCreatedPageSelfUrl | `outputs('RL11_Compose_Out_Created_Page_Self_Url')` |
| `outfinaltargetsectionpagesurl` | OutFinalTargetSectionPagesUrl | `outputs('RL12_Compose_Out_Final_Target_Section_Pages_Url')` |
| `outresolverresult` | OutResolverResult | `outputs('RL13_Compose_Out_Resolver_Result')` |
| `outexistingpageselfurl` | OutExistingPageSelfUrl | `outputs('RL14_Compose_Out_Existing_Page_Self_Url')` |
| `outpagedecision` | OutPageDecision | `outputs('RL15_Compose_Out_Page_Decision')` |
| `outpageroute` | OutPageRoute | `outputs('RL16_Compose_Out_Page_Route')` |
| `outpageaction` | OutPageAction | `outputs('RL17_Compose_Out_Page_Action')` |
| `outupdatehtmlfragment` | OutUpdateHtmlFragment | `outputs('RL18_Compose_Out_Update_Html_Fragment')` |
| `outagentresponsesummary` | OutAgentResponseSummary | `outputs('RL19_Compose_Out_Agent_Response_Summary')` |
| `outstatus` | OutStatus | `outputs('RL20_Compose_Out_Status')` |

12. Peek Code the Respond action. Confirm all 20 keys present, correct titles, correct expressions.

---

## Step 5 — Scope health check and publish

1. Peek Code `Scope_FlowB_Parent` (the outer Scope). This is the full-flow Peek Code — paste to GitHub as `scope-peek-codes/scope-parent.md`.
2. Flow Checker — 0 errors required before saving.
3. Save draft.
4. Close the flow, wait 10s, reopen. Wait 20s untouched.
5. Flow Checker again — 0 errors.
6. Peek Code `Scope_FlowB_Parent` again — cross-reference against the version you just pushed to GitHub.
7. Publish.
8. Push scope peek code to `flows/flow-b-v2/scope-peek-codes/scope-parent.md`.

---

## Parent known-good values (populate after Step 5)

Create `flows/flow-b-v2/parent/known-good-values.md` and record every expression from Steps 3–4 once confirmed green. Minimum entries: RT01, RT02 condition, RT03a/b input mappings, RL20 (the outstatus coalesce with ERROR fallback), RL_Respond key list.
