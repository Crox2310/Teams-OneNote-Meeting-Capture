# Flow B v2 One-Off Child — Build Instructions

**Flow name:** `PA - Meeting Capture - B - OneOff - v2`
**Build methodology:** Compose-based state passing. No SetVariable. No variables.
**Session-start check:** open flow, wait 20s, Flow Checker, Peek Code `Scope_FlowB_OneOff`.

---

## Prerequisites

1. Read `flows/flow-b-v2/oneoff/trigger-contract.md` — the 6 inputs and the key-remap warning.
2. Read `flows/flow-b-v2/oneoff/output-contract.md` — the 16 output fields.
3. This flow must be **solution-aware** and end with a **"Respond to a PowerApp or Flow"** action for the parent's Run-a-Child-Flow to work.

---

## Step 1 — Create the flow

1. Create a new flow.
2. Trigger: **PowerApps (V2)** — "When a Power App or flow calls this flow".
3. Name: `PA - Meeting Capture - B - OneOff - v2`.
4. Add the 6 input fields **in order**:

| Title | Type | Required |
|---|---|---|
| MeetingTitle | Text | Yes |
| MeetingId | Text | Yes |
| OccurrenceDate | Text | Yes |
| PageHtml | Text | Yes |
| EndTime | Text | No |
| JoinUrl | Text | No |

5. Peek Code the trigger. Confirm: `text`=MeetingTitle, `text_1`=MeetingId, `text_2`=OccurrenceDate, `text_3`=PageHtml, `text_4`=EndTime, `text_5`=JoinUrl.

---

## Step 2 — Outer Scope

1. Add a **Scope**.
2. Name: `Scope_FlowB_OneOff`
3. Description: `Full one-off child — Peek Code this for a full-flow health check`
4. All subsequent actions go inside this Scope.

---

## Step 3 — Scope_Normalize (NZ)

1. Add a **Scope** inside `Scope_FlowB_OneOff`.
2. Name: `Scope_Normalize`
3. Description: `Re-expose trigger inputs as named Composes, then derive page title and HTML fragment`

### Input Composes (NZ00a–NZ00f)
Add 6 **Compose** actions. These are the single source of truth for all trigger values — nothing downstream reads `triggerBody()?['text_N']` directly.

| Ref | Name | Expression |
|---|---|---|
| NZ00a | `NZ00a Compose MeetingTitle` | `triggerBody()?['text']` |
| NZ00b | `NZ00b Compose MeetingId` | `triggerBody()?['text_1']` |
| NZ00c | `NZ00c Compose OccurrenceDate` | `triggerBody()?['text_2']` |
| NZ00d | `NZ00d Compose PageHtml` | `triggerBody()?['text_3']` |
| NZ00e | `NZ00e Compose EndTime` | `triggerBody()?['text_4']` |
| NZ00f | `NZ00f Compose JoinUrl` | `triggerBody()?['text_5']` |

### NZ01 — Safe Page Title

```
if(empty(trim(coalesce(outputs('NZ00a_Compose_MeetingTitle'), ''))), 'Untitled Meeting', concat(substring(replace(replace(replace(replace(outputs('NZ00a_Compose_MeetingTitle'), '&', 'and'), '<', ''), '>', ''), '"', ''), 0, min(150, length(replace(replace(replace(replace(outputs('NZ00a_Compose_MeetingTitle'), '&', 'and'), '<', ''), '>', ''), '"', '')))), if(empty(coalesce(outputs('NZ00c_Compose_OccurrenceDate'), '')), '', concat(' - ', formatDateTime(outputs('NZ00c_Compose_OccurrenceDate'), 'd MMM yyyy')))))
```

Ref: `NZ01 Compose Safe Page Title`

### NZ02 — Update Html Fragment

```
concat('<hr><h2>Automated update</h2><p><strong>Updated by:</strong> Meeting Capture Agent</p><p><strong>Update note:</strong> Meeting details were refreshed by the automation. Existing human-entered notes were preserved below.</p>', outputs('NZ00d_Compose_PageHtml'))
```

Ref: `NZ02 Compose Update Html Fragment`

### NZ03 — Target Section Pages Url (FIXED — text literal, not Expression)

4. Add a **Compose**.
5. Name: `NZ03 Compose Target Section Pages Url`
6. Inputs: switch to **Value tab** (not Expression). Paste the literal URL:
```
https://www.onenote.com/api/v1.0/myOrganization/siteCollections/b5f8860c-4772-4e8b-b340-e80ba9d490fa/sites/d814850f-59bb-4182-92b7-e25d8c6a0487/notes/sections/1-cbb5e863-2bd7-458f-9944-4fec9d20607b/pages
```
7. Peek Code. Confirm inputs is the literal URL string with no `@` prefix.

---

## Step 4 — Scope_MappingLookup (ML)

1. Add a **Scope** after `Scope_Normalize`.
2. Name: `Scope_MappingLookup`
3. Description: `Look up existing mapping row for this MeetingId + OccurrenceDate`

### ML01 — Get items (SharePoint connector)

4. Add a **SharePoint — Get items** action.
5. Name: `ML01 Get items`
6. Site Address: `https://jsainsbury.sharepoint.com/sites/coplt`
7. List Name: select `RecurringMeetingSectionMap` (GUID `186b3c9f-e758-4e85-83d5-685946614a0a`)
8. In Advanced parameters, set **Top Count**: `500`
9. Peek Code. Confirm dataset URL, table GUID, `$top: 500`.

### ML02 — Filter Existing Mapping OneOff (Filter Array)

10. Add a **Filter Array** action (Data Operations).
11. Name: `ML02 Filter Existing Mapping OneOff`
12. From (Expression tab): `body('ML01_Get_items')?['value']`
13. Filter condition:
 - Left (Expression): `item()?['MeetingId']`
 - Operator: `is equal to`
 - Right (Expression): `outputs('NZ00b_Compose_MeetingId')`
14. Add a second condition (And):
 - Left (Expression): `item()?['OccurrenceDate']`
 - Operator: `is equal to`
 - Right (Expression): `outputs('NZ00c_Compose_OccurrenceDate')`
15. Peek Code. Confirm from and both where clauses.

### ML03 — Compose Match Count

16. Add a **Compose**.
17. Name: `ML03 Compose Match Count`
18. Expression: `length(body('ML02_Filter_Existing_Mapping_OneOff'))`
19. Peek Code.

### ML04 — Compose Mapping Exists

20. Add a **Compose**.
21. Name: `ML04 Compose Mapping Exists`
22. Expression: `greater(outputs('ML03_Compose_Match_Count'), 0)`
23. Peek Code.

---

## Step 5 — Mapping branch Condition

1. After `Scope_MappingLookup`, add a **Condition**.
2. Name: `Condition Mapping Exists`
3. Left (Expression): `outputs('ML04_Compose_Mapping_Exists')`
4. Operator: `is equal to`
5. Right (Expression): `true`

---

## Step 6 — Scope_ExistingMapping (EM) — True branch

1. In the **True** branch, add a **Scope**.
2. Name: `Scope_ExistingMapping`
3. Description: `Read the existing mapping row — no section resolution needed (fixed section)`

### EM01 — Compose Existing Page Self Url

4. Add a **Compose**.
5. Name: `EM01 Compose Existing Page Self Url`
6. Expression:
```
coalesce(first(body('ML02_Filter_Existing_Mapping_OneOff'))?['PageSelfUrl'], '')
```

### EM02 — Compose Existing Page Web Url

7. Add a **Compose**.
8. Name: `EM02 Compose Existing Page Web Url`
9. Expression:
```
coalesce(first(body('ML02_Filter_Existing_Mapping_OneOff'))?['PageWebUrl'], '')
```

### EM03 — Compose Existing Row Id

10. Add a **Compose**.
11. Name: `EM03 Compose Existing Row Id`
12. Expression:
```
string(first(body('ML02_Filter_Existing_Mapping_OneOff'))?['ID'])
```

---

## Step 7 — Scope_NewMapping (NM) — False branch

1. In the **False** branch, add a **Scope**.
2. Name: `Scope_NewMapping`
3. Description: `Create a new mapping row — section is fixed, no resolution needed`

### NM01 — Create Mapping Item OneOff (SharePoint connector)

4. Add a **SharePoint — Create item** action.
5. Name: `NM01 Create Mapping Item OneOff`
6. Site Address: `https://jsainsbury.sharepoint.com/sites/coplt`
7. List Name: `RecurringMeetingSectionMap`
8. Set fields (all Expression tab unless marked):

| Field | Value |
|---|---|
| Title | `Mapping` (text literal) |
| MeetingTitle | `outputs('NZ00a_Compose_MeetingTitle')` |
| MeetingId | `outputs('NZ00b_Compose_MeetingId')` |
| OccurrenceDate | `outputs('NZ00c_Compose_OccurrenceDate')` |
| SectionPagesUrl | `outputs('NZ03_Compose_Target_Section_Pages_Url')` |
| JoinUrl | `trim(coalesce(outputs('NZ00f_Compose_JoinUrl'), ''))` |
| EndTime | `coalesce(outputs('NZ00e_Compose_EndTime'), '')` |
| Status | `Active` (text literal) |

9. Peek Code. Confirm all field values.

### NM02 — Compose New Row Id

10. Add a **Compose**.
11. Name: `NM02 Compose New Row Id`
12. Expression:
```
string(outputs('NM01_Create_Mapping_Item_OneOff')?['body/ID'])
```

### NM03 — Compose Mapping Write Succeeded

13. Add a **Compose**.
14. Name: `NM03 Compose Mapping Write Succeeded`
15. Expression:
```
if(equals(outputs('NM01_Create_Mapping_Item_OneOff')?['statusCode'], 201), 'true', 'false')
```

---

## Step 8 — Scope_PageResolve (PG)

After the Condition (outside both branches, still inside `Scope_FlowB_OneOff`):

1. Add a **Scope**.
2. Name: `Scope_PageResolve`
3. Description: `S1 duplicate-page guard — filter by title, then append or create`

### PG01 — Get Pages In Section (OneNote connector)

4. Add a **OneNote — Get pages in section** action.
5. Name: `PG01 Get Pages In Section`
6. Notebook Key: `Meeting Notes|$|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Meeting Notes`
7. Section ID (Expression tab): `outputs('NZ03_Compose_Target_Section_Pages_Url')`
8. Peek Code. Confirm notebookKey string and sectionId expression.

### PG02 — Filter Pages By Title (Filter Array)

9. Add a **Filter Array** action.
10. Name: `PG02 Filter Pages By Title`
11. From (Expression): `outputs('PG01_Get_Pages_In_Section')?['body']?['value']`
12. Condition:
 - Left (Expression): `item()?['title']`
 - Operator: `contains`
 - Right (Expression): `formatDateTime(outputs('NZ00c_Compose_OccurrenceDate'), 'd MMM yyyy')`
13. Peek Code.

### PG03 — Compose Page Match Count

14. Add a **Compose**.
15. Name: `PG03 Compose Page Match Count`
16. Expression: `length(body('PG02_Filter_Pages_By_Title'))`

### PG04 — Compose Page Decision

17. Add a **Compose**.
18. Name: `PG04 Compose Page Decision`
19. Expression:
```
if(greater(outputs('PG03_Compose_Page_Match_Count'), 0), 'PAGE_EXISTS', 'PAGE_NOT_FOUND')
```

### PG05 — Condition Page Exists

20. Add a **Condition**.
21. Name: `PG05 Condition Page Exists`
22. Left (Expression): `outputs('PG04_Compose_Page_Decision')`
23. Operator: `is equal to`
24. Right: `PAGE_EXISTS`

#### True branch — Append existing page

##### PG06 — Compose Existing Page Id
25. Add a **Compose**.
26. Name: `PG06 Compose Existing Page Id`
27. Expression:
```
if(greater(length(body('PG02_Filter_Pages_By_Title')), 0), first(body('PG02_Filter_Pages_By_Title'))?['id'], '')
```

##### PG07 — Update Page Content (OneNote connector)
28. Add a **OneNote — Update page content** action.
29. Name: `PG07 Update Page Content`
30. Fields:
 - Notebook Key: `Meeting Notes|$|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Meeting Notes`
 - Section ID (Expression): `outputs('NZ03_Compose_Target_Section_Pages_Url')`
 - Page ID (Expression): `outputs('PG06_Compose_Existing_Page_Id')`
 - Target: `body`
 - Action: `append`
 - Content (Expression): `outputs('NZ02_Compose_Update_Html_Fragment')`
31. Peek Code.

##### PG08 — Compose Page Action (Append)
32. Add a **Compose**.
33. Name: `PG08 Compose Page Action`
34. Inputs (Value tab — text literal): `UpdatedAppend`

##### PG09 — Compose Created Page Link (Append path)
35. Add a **Compose**.
36. Name: `PG09 Compose Created Page Link`
37. Expression:
```
coalesce(outputs('EM02_Compose_Existing_Page_Web_Url'), '')
```

##### PG10 — Compose Created Page Self Url (Append path)
38. Add a **Compose**.
39. Name: `PG10 Compose Created Page Self Url`
40. Expression:
```
coalesce(outputs('EM01_Compose_Existing_Page_Self_Url'), '')
```

#### False branch — Create new page

##### PG11 — Create Page In Section (OneNote connector)
41. Add a **OneNote — Create page in section** action.
42. Name: `PG11 Create Page In Section`
43. Fields:
 - Notebook Key: `Meeting Notes|$|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Meeting Notes`
 - Section ID (Expression): `outputs('NZ03_Compose_Target_Section_Pages_Url')`
 - Page Content (Expression): `outputs('NZ00d_Compose_PageHtml')`
44. Peek Code.

##### PG12 — Compose Page Action (Create)
45. Add a **Compose**.
46. Name: `PG12 Compose Page Action`
47. Inputs (Value tab): `Created`

##### PG13 — Compose Created Page Link (Create path)
48. Add a **Compose**.
49. Name: `PG13 Compose Created Page Link`
50. Expression:
```
coalesce(outputs('PG11_Create_Page_In_Section')?['body']?['links']?['oneNoteWebUrl']?['href'], '')
```

##### PG14 — Compose Created Page Self Url (Create path)
51. Add a **Compose**.
52. Name: `PG14 Compose Created Page Self Url`
53. Expression:
```
coalesce(outputs('PG11_Create_Page_In_Section')?['body']?['self'], '')
```

### PG15–PG17 — Coalesce page outputs (after both branches)

54. After the PG05 Condition (still inside `Scope_PageResolve`), add 3 **Compose** actions:

| Ref | Name | Expression |
|---|---|---|
| PG15 | `PG15 Compose Page Action Final` | `coalesce(outputs('PG08_Compose_Page_Action'), outputs('PG12_Compose_Page_Action'), '')` |
| PG16 | `PG16 Compose Page Link Final` | `coalesce(outputs('PG09_Compose_Created_Page_Link'), outputs('PG13_Compose_Created_Page_Link'), '')` |
| PG17 | `PG17 Compose Page Self Url Final` | `coalesce(outputs('PG10_Compose_Created_Page_Self_Url'), outputs('PG14_Compose_Created_Page_Self_Url'), '')` |

---

## Step 9 — Scope_WriteBack (WB)

Runs when `PageAction == 'Created'` only.

1. Add a **Scope** after `Scope_PageResolve`.
2. Name: `Scope_WriteBack`
3. Description: `MERGE PageSelfUrl and PageWebUrl back to the mapping row — new page only`
4. Set runAfter on `Scope_WriteBack`: after `Scope_PageResolve`, condition `Succeeded`.

### WB00 — Compose Target Row Id

5. Add a **Compose**.
6. Name: `WB00 Compose Target Row Id`
7. Expression:
```
coalesce(outputs('NM02_Compose_New_Row_Id'), outputs('EM03_Compose_Existing_Row_Id'), '')
```

### WB01 — Condition Is New Page

8. Add a **Condition**.
9. Name: `WB01 Condition Is New Page`
10. Left (Expression): `outputs('PG15_Compose_Page_Action_Final')`
11. Operator: `is equal to`
12. Right: `Created`

#### True branch only — HTTP MERGE

##### WB02 — HTTP Update SP Page Urls
13. Add an **HTTP** action (must use the HTTP connector, not Send HTTP request).
14. Name: `WB02 HTTP Update SP Page Urls`
15. Method: `PATCH`
16. URI (Expression):
```
concat('https://jsainsbury.sharepoint.com/sites/coplt/_api/lists(''186b3c9f-e758-4e85-83d5-685946614a0a'')/items(', outputs('WB00_Compose_Target_Row_Id'), ')')
```
17. Headers:
 - `Content-Type`: `application/json;odata=verbose`
 - `IF-MATCH`: `*`
 - `X-HTTP-Method`: `MERGE`
18. Body (Expression):
```
concat('{"__metadata":{"type":"SP.Data.RecurringMeetingSectionMapListItem"},"PageSelfUrl":"', outputs('PG17_Compose_Page_Self_Url_Final'), '","PageWebUrl":"', outputs('PG16_Compose_Page_Link_Final'), '"}')
```
19. Peek Code. Confirm method, URI, headers, body expression.

---

## Step 10 — Scope_Status (ST)

1. Add a **Scope** after `Scope_WriteBack`.
2. Name: `Scope_Status`
3. Description: `Derive outstatus and outagentresponsesummary`

### ST01 — Compose Out Status

4. Add a **Compose**.
5. Name: `ST01 Compose Out Status`
6. Expression:
```
if(and(contains(createArray('Created','Updated','UpdatedAppend'), outputs('PG15_Compose_Page_Action_Final')), equals(coalesce(outputs('NM03_Compose_Mapping_Write_Succeeded'), 'true'), 'true')), 'SUCCESS', if(and(contains(createArray('Created','Updated','UpdatedAppend'), outputs('PG15_Compose_Page_Action_Final')), equals(coalesce(outputs('NM03_Compose_Mapping_Write_Succeeded'), 'true'), 'false')), 'PARTIAL_SUCCESS', 'ERROR'))
```
7. Peek Code. Verify parenthesis balance before saving.

### ST02 — Compose Agent Response Summary

8. Add a **Compose**.
9. Name: `ST02 Compose Agent Response Summary`
10. **⚠ Pull the expression from live v1 `Compose_AgentResponseSummary` — not in the known-good reference.** Paste the live expression here. If v1's expression references variable names, remap to the equivalent Compose outputs in v2.

---

## Step 11 — Scope_Respond (RP)

1. Add a **Scope** after `Scope_Status`.
2. Name: `Scope_Respond`
3. Description: `Compose 16 output fields and respond`

### RP01–RP16 — Output Composes

4. Add 16 **Compose** actions:

| Ref | Name | Expression |
|---|---|---|
| RP01 | `RP01 Compose outspitemcount` | `int(coalesce(outputs('ML03_Compose_Match_Count'), 0))` |
| RP02 | `RP02 Compose outmatchcount` | `string(outputs('ML03_Compose_Match_Count'))` |
| RP03 | `RP03 Compose outbranchresult` | `string(outputs('ML03_Compose_Match_Count'))` |
| RP04 | `RP04 Compose outonenoteresolverresult` | `''` (text literal — fixed section, no resolver) |
| RP05 | `RP05 Compose outtargetsectionpagesurl` | `outputs('NZ03_Compose_Target_Section_Pages_Url')` |
| RP06 | `RP06 Compose outcreatedpagelink` | `outputs('PG16_Compose_Page_Link_Final')` |
| RP07 | `RP07 Compose outcreatedpageselfurl` | `outputs('PG17_Compose_Page_Self_Url_Final')` |
| RP08 | `RP08 Compose outfinaltargetsectionpagesurl` | `outputs('NZ03_Compose_Target_Section_Pages_Url')` |
| RP09 | `RP09 Compose outresolverresult` | `''` (text literal) |
| RP10 | `RP10 Compose outexistingpageselfurl` | `coalesce(outputs('EM01_Compose_Existing_Page_Self_Url'), '')` |
| RP11 | `RP11 Compose outpagedecision` | `outputs('PG04_Compose_Page_Decision')` |
| RP12 | `RP12 Compose outpageroute` | `string(equals(outputs('PG04_Compose_Page_Decision'), 'PAGE_EXISTS'))` |
| RP13 | `RP13 Compose outpageaction` | `outputs('PG15_Compose_Page_Action_Final')` |
| RP14 | `RP14 Compose outupdatehtmlfragment` | `outputs('NZ02_Compose_Update_Html_Fragment')` |
| RP15 | `RP15 Compose outagentresponsesummary` | `outputs('ST02_Compose_Agent_Response_Summary')` |
| RP16 | `RP16 Compose outstatus` | `outputs('ST01_Compose_Out_Status')` |

### RP_Respond — Respond to a PowerApp or Flow

5. Add a **"Respond to a PowerApp or Flow"** action.
6. Name: `RP_Respond`
7. Add all 16 output fields binding to RP01–RP16:

| Key | Title | Value (Expression) |
|---|---|---|
| `outspitemcount` | OutSPItemCount | `outputs('RP01_Compose_outspitemcount')` |
| `outmatchcount` | OutMatchCount | `outputs('RP02_Compose_outmatchcount')` |
| `outbranchresult` | OutBranchResult | `outputs('RP03_Compose_outbranchresult')` |
| `outonenoteresolverresult` | OutOneNoteResolverResult | `outputs('RP04_Compose_outonenoteresolverresult')` |
| `outtargetsectionpagesurl` | OutTargetSectionPagesUrl | `outputs('RP05_Compose_outtargetsectionpagesurl')` |
| `outcreatedpagelink` | OutCreatedPageLink | `outputs('RP06_Compose_outcreatedpagelink')` |
| `outcreatedpageselfurl` | OutCreatedPageSelfUrl | `outputs('RP07_Compose_outcreatedpageselfurl')` |
| `outfinaltargetsectionpagesurl` | OutFinalTargetSectionPagesUrl | `outputs('RP08_Compose_outfinaltargetsectionpagesurl')` |
| `outresolverresult` | OutResolverResult | `outputs('RP09_Compose_outresolverresult')` |
| `outexistingpageselfurl` | OutExistingPageSelfUrl | `outputs('RP10_Compose_outexistingpageselfurl')` |
| `outpagedecision` | OutPageDecision | `outputs('RP11_Compose_outpagedecision')` |
| `outpageroute` | OutPageRoute | `outputs('RP12_Compose_outpageroute')` |
| `outpageaction` | OutPageAction | `outputs('RP13_Compose_outpageaction')` |
| `outupdatehtmlfragment` | OutUpdateHtmlFragment | `outputs('RP14_Compose_outupdatehtmlfragment')` |
| `outagentresponsesummary` | OutAgentResponseSummary | `outputs('RP15_Compose_outagentresponsesummary')` |
| `outstatus` | OutStatus | `outputs('RP16_Compose_outstatus')` |

8. Peek Code the Respond action. Confirm all 16 keys and expressions.

---

## Step 12 — Scope health check, publish, GitHub push

1. Peek Code `Scope_FlowB_OneOff` (outer Scope) — full-flow health check.
2. Flow Checker — 0 errors.
3. Save draft. Close, wait 10s, reopen, wait 20s.
4. Flow Checker again.
5. Peek Code outer Scope again — cross-reference.
6. Publish.
7. Set connectors in **Run only users**: Teams, SharePoint, OneNote — "Use this connection (david.croxson@sainsburys.co.uk)".
8. Push outer Scope Peek Code to `flows/flow-b-v2/scope-peek-codes/scope-oneoff.md`.
9. Push known-good values to `flows/flow-b-v2/oneoff/known-good-values.md`.

---

## Run-only-users checklist (required for parent's Run-a-Child-Flow)

- Open the flow on `make.powerautomate.com`
- Click the three-dot menu → Edit → Run only users
- For each connector (SharePoint, OneNote): set to **"Use this connection (david.croxson@sainsburys.co.uk)"**
- Save
