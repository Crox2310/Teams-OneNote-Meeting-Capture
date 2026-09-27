# Flow B v2 Recurring Child — Build Instructions

**Flow name:** `PA - Meeting Capture - B - Recurring - v2`
**Build methodology:** Compose-based state passing. No SetVariable. No variables.
**Session-start check:** open flow, wait 20s, Flow Checker, Peek Code `Scope_FlowB_Recurring`.

---

## Prerequisites

1. Read `flows/flow-b-v2/recurring/trigger-contract.md` — the 6 inputs and the key-remap warning.
2. Read `flows/flow-b-v2/recurring/output-contract.md` — the 16 output fields and OutStatus precedence.
3. PowerAppV2 trigger + terminal "Respond to a PowerApp or Flow" + "Use this connection" in Run-only-users + solution-aware.

---

## Step 1 — Create the flow

1. Create a new flow.
2. Trigger: **PowerApps (V2)** — "When a Power App or flow calls this flow".
3. Name: `PA - Meeting Capture - B - Recurring - v2`.
4. Add the 6 input fields **in order**:

| Title | Type | Required |
|---|---|---|
| MeetingTitle | Text | Yes |
| SeriesMasterId | Text | Yes |
| OccurrenceDate | Text | Yes |
| PageHtml | Text | Yes |
| EndTime | Text | No |
| JoinUrl | Text | No |

5. Peek Code the trigger. Confirm: `text`=MeetingTitle, `text_1`=SeriesMasterId, `text_2`=OccurrenceDate, `text_3`=PageHtml, `text_4`=EndTime, `text_5`=JoinUrl.

---

## Step 2 — Outer Scope

1. Add a **Scope**.
2. Name: `Scope_FlowB_Recurring`
3. Description: `Full recurring child — Peek Code this for a full-flow health check`

---

## Step 3 — Scope_Normalize (NZ)

1. Add a **Scope** inside `Scope_FlowB_Recurring`.
2. Name: `Scope_Normalize`

### Input Composes (NZ00a–NZ00f)

| Ref | Name | Expression |
|---|---|---|
| NZ00a | `NZ00a Compose MeetingTitle` | `triggerBody()?['text']` |
| NZ00b | `NZ00b Compose SeriesMasterId` | `triggerBody()?['text_1']` |
| NZ00c | `NZ00c Compose OccurrenceDate` | `triggerBody()?['text_2']` |
| NZ00d | `NZ00d Compose PageHtml` | `triggerBody()?['text_3']` |
| NZ00e | `NZ00e Compose EndTime` | `triggerBody()?['text_4']` |
| NZ00f | `NZ00f Compose JoinUrl` | `triggerBody()?['text_5']` |

### NZ01 — Compose Section Display Name
Ref: `NZ01 Compose Section Display Name`
Expression: `coalesce(outputs('NZ00a_Compose_MeetingTitle'), '')`

### NZ02 — Compose Safe Section Name
Ref: `NZ02 Compose Safe Section Name`
```
if(empty(trim(coalesce(outputs('NZ01_Compose_Section_Display_Name'), ''))), 'Mtg - Untitled Meeting', concat('Mtg - ', substring(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(outputs('NZ01_Compose_Section_Display_Name'), '/', '-'), ':', '-'), '&', 'and'), '?', ''), '*', ''), '<', ''), '>', ''), '"', ''), '|', ''), '#', ''), '''', ''), '%', ''), '~', ''), 0, min(43, length(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(outputs('NZ01_Compose_Section_Display_Name'), '/', '-'), ':', '-'), '&', 'and'), '?', ''), '*', ''), '<', ''), '>', ''), '"', ''), '|', ''), '#', ''), '''', ''), '%', ''), '~', ''))))))
```

### NZ03 — Compose Safe Page Title
Ref: `NZ03 Compose Safe Page Title`
```
if(empty(trim(coalesce(outputs('NZ00a_Compose_MeetingTitle'), ''))), 'Untitled Meeting', concat(substring(replace(replace(replace(replace(outputs('NZ00a_Compose_MeetingTitle'), '&', 'and'), '<', ''), '>', ''), '"', ''), 0, min(150, length(replace(replace(replace(replace(outputs('NZ00a_Compose_MeetingTitle'), '&', 'and'), '<', ''), '>', ''), '"', '')))), if(empty(coalesce(outputs('NZ00c_Compose_OccurrenceDate'), '')), '', concat(' - ', formatDateTime(outputs('NZ00c_Compose_OccurrenceDate'), 'd MMM yyyy')))))
```

### NZ04 — Compose Update Html Fragment
Ref: `NZ04 Compose Update Html Fragment`
```
concat('<hr><h2>Automated update</h2><p><strong>Updated by:</strong> Meeting Capture Agent</p><p><strong>Update note:</strong> Meeting details were refreshed by the automation. Existing human-entered notes were preserved below.</p>', outputs('NZ00d_Compose_PageHtml'))
```

Peek Code all NZ actions before proceeding.

---

## Step 4 — Scope_MappingLookup (ML)

1. Add a **Scope** after `Scope_Normalize`.
2. Name: `Scope_MappingLookup`

### ML01 — Get items (SharePoint connector)
4. Add **SharePoint — Get items**. Name: `ML01 Get items`
5. Site: `https://jsainsbury.sharepoint.com/sites/coplt` · List: `RecurringMeetingSectionMap` (GUID `186b3c9f-e758-4e85-83d5-685946614a0a`) · Top Count: `500`

### ML02 — Filter Existing Mapping (Filter Array)
6. Add **Filter Array**. Name: `ML02 Filter Existing Mapping`
7. From (Expression): `body('ML01_Get_items')?['value']`
8. Condition 1: Left `item()?['SeriesMasterId']` · is equal to · Right `outputs('NZ00b_Compose_SeriesMasterId')`
9. Condition 2 (And): Left `item()?['OccurrenceDate']` · is equal to · Right `outputs('NZ00c_Compose_OccurrenceDate')`

### ML03 — Compose Match Count
Ref: `ML03 Compose Match Count` · Expression: `length(body('ML02_Filter_Existing_Mapping'))`

### ML04 — Compose Mapping Exists
Ref: `ML04 Compose Mapping Exists` · Expression: `greater(outputs('ML03_Compose_Match_Count'), 0)`

Peek Code all ML actions.

---

## Step 5 — Mapping branch Condition

1. After `Scope_MappingLookup`, add a **Condition**.
2. Name: `Condition Mapping Exists`
3. Left (Expression): `outputs('ML04_Compose_Mapping_Exists')` · is equal to · Right: `true`

---

## Step 6 — Scope_ExistingMapping (EM) — True branch

1. In the **True** branch, add a **Scope**. Name: `Scope_ExistingMapping`

### EM01 — Compose Is Stale Row
Ref: `EM01 Compose Is Stale Row`
Expression:
```
empty(coalesce(first(body('ML02_Filter_Existing_Mapping'))?['SectionPagesUrl'], ''))
```

### EM02 — Compose Existing Section Pages Url
Ref: `EM02 Compose Existing Section Pages Url`
Expression:
```
coalesce(first(body('ML02_Filter_Existing_Mapping'))?['SectionPagesUrl'], '')
```

### EM03 — Compose Existing Page Self Url
Ref: `EM03 Compose Existing Page Self Url`
Expression:
```
coalesce(first(body('ML02_Filter_Existing_Mapping'))?['PageSelfUrl'], '')
```

### EM04 — Compose Existing Page Web Url
Ref: `EM04 Compose Existing Page Web Url`
Expression:
```
coalesce(first(body('ML02_Filter_Existing_Mapping'))?['PageWebUrl'], '')
```

### EM05 — Compose Existing Row Id
Ref: `EM05 Compose Existing Row Id`
Expression:
```
string(first(body('ML02_Filter_Existing_Mapping'))?['ID'])
```

### EM06 — Stale row condition
1. Add a **Condition**. Name: `EM06 Condition Is Stale Row`
2. Left (Expression): `outputs('EM01_Compose_Is_Stale_Row')` · is equal to · Right: `true`
3. **True branch:** leave empty — stale rows skip all page work and fall through to Status which emits `STALE_MAPPING`.
4. **False branch:** leave empty — non-stale rows continue to `Scope_PageResolve` which runs after this condition.

Peek Code all EM actions.

---

## Step 7 — Scope_NewMapping (NM) — False branch

1. In the **False** branch of `Condition Mapping Exists`, add a **Scope**. Name: `Scope_NewMapping`

### NM01 — Get Sections (OneNote connector)
2. Add **OneNote — Get sections in notebook**. Name: `NM01 Get Sections`
3. Notebook Key: `Meeting Notes|$|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Meeting Notes`

### NM02 — Filter Section By Name (Filter Array)
4. Add **Filter Array**. Name: `NM02 Filter Section By Name`
5. From (Expression): `outputs('NM01_Get_Sections')?['body/value']`
6. Condition: Left `item()?['name']` · is equal to · Right `outputs('NZ02_Compose_Safe_Section_Name')`

### NM03 — Compose Section Match Count
Ref: `NM03 Compose Section Match Count` · Expression: `length(body('NM02_Filter_Section_By_Name'))`

### NM04 — Section branch Condition
7. Add a **Condition**. Name: `NM04 Condition Section Exists`
8. Left (Expression): `outputs('NM03_Compose_Section_Match_Count')` · is equal to · Right: `0`

#### True branch (count=0) — Create Section

##### NM05a — Create Section (OneNote connector)
9. Add **OneNote — Create section in notebook**. Name: `NM05a Create Section`
10. Notebook Key: `Meeting Notes|$|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Meeting Notes`
11. Section name (Expression): `outputs('NZ02_Compose_Safe_Section_Name')`

##### NM05b — Compose Target Section Pages Url (Created)
12. Add **Compose**. Name: `NM05b Compose Target Section Pages Url`
13. Expression: `outputs('NM05a_Create_Section')?['body']?['pagesUrl']`

##### NM05c — Compose Resolver Result (Created)
14. Add **Compose**. Name: `NM05c Compose Resolver Result`
15. Inputs (Value tab — literal): `CreatedSection`

#### False branch (count>0) — Use existing

##### NM06a — Compose Target Section Pages Url (Existing)
16. Add **Compose**. Name: `NM06a Compose Target Section Pages Url`
17. Expression: `first(body('NM02_Filter_Section_By_Name'))?['pagesUrl']`

##### NM06b — Compose Resolver Result (Existing)
18. Add **Compose**. Name: `NM06b Compose Resolver Result`
19. Inputs (Value tab — literal): `ExistingSection`

### NM07 — Compose Target Section Pages Url Final (after NM04 condition)
20. Add **Compose**. Name: `NM07 Compose Target Section Pages Url Final`
21. Expression:
```
coalesce(outputs('NM05b_Compose_Target_Section_Pages_Url'), outputs('NM06a_Compose_Target_Section_Pages_Url'), '')
```

### NM08 — Compose Resolver Result Final (after NM04 condition)
22. Add **Compose**. Name: `NM08 Compose Resolver Result Final`
23. Expression:
```
coalesce(outputs('NM05c_Compose_Resolver_Result'), outputs('NM06b_Compose_Resolver_Result'), '')
```

### NM09 — Create Mapping Item Recurring (SharePoint connector)
24. Add **SharePoint — Create item**. Name: `NM09 Create Mapping Item Recurring`
25. Site: `https://jsainsbury.sharepoint.com/sites/coplt` · List: `RecurringMeetingSectionMap`
26. Fields:

| Field | Value |
|---|---|
| Title | `Mapping` (literal) |
| MeetingTitle | `outputs('NZ00a_Compose_MeetingTitle')` |
| SeriesMasterId | `outputs('NZ00b_Compose_SeriesMasterId')` |
| OccurrenceDate | `outputs('NZ00c_Compose_OccurrenceDate')` |
| SectionPagesUrl | `outputs('NM07_Compose_Target_Section_Pages_Url_Final')` |
| JoinUrl | `trim(coalesce(outputs('NZ00f_Compose_JoinUrl'), ''))` |
| EndTime | `coalesce(outputs('NZ00e_Compose_EndTime'), '')` |
| Status | `Active` (literal) |

### NM10 — Compose New Row Id
27. Add **Compose**. Name: `NM10 Compose New Row Id`
28. Expression: `string(outputs('NM09_Create_Mapping_Item_Recurring')?['body/ID'])`

### NM11 — Compose Mapping Write Succeeded
29. Add **Compose**. Name: `NM11 Compose Mapping Write Succeeded`
30. Expression: `if(equals(outputs('NM09_Create_Mapping_Item_Recurring')?['statusCode'], 201), 'true', 'false')`

Peek Code all NM actions.

---

## Step 8 — Scope_PageResolve (PG)

After the `Condition Mapping Exists` (outside both branches, still inside `Scope_FlowB_Recurring`).

1. Add a **Scope**. Name: `Scope_PageResolve`
2. Description: `S1 duplicate-page guard — filter by title, then append or create. Every branch records outpageaction.`

### PG00 — Compose Target Section Pages Url
3. Add **Compose**. Name: `PG00 Compose Target Section Pages Url`
4. Expression:
```
coalesce(outputs('EM02_Compose_Existing_Section_Pages_Url'), outputs('NM07_Compose_Target_Section_Pages_Url_Final'), '')
```

### PG01 — Get Pages In Section (OneNote connector)
5. Add **OneNote — Get pages in section**. Name: `PG01 Get Pages In Section`
6. Notebook Key: `Meeting Notes|$|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Meeting Notes`
7. Section ID (Expression): `outputs('PG00_Compose_Target_Section_Pages_Url')`

### PG02 — Filter Pages By Title (Filter Array)
8. Add **Filter Array**. Name: `PG02 Filter Pages By Title`
9. From (Expression): `outputs('PG01_Get_Pages_In_Section')?['body']?['value']`
10. Condition: Left `item()?['title']` · contains · Right `formatDateTime(outputs('NZ00c_Compose_OccurrenceDate'), 'd MMM yyyy')`

### PG03 — Compose Page Match Count
Ref: `PG03 Compose Page Match Count` · Expression: `length(body('PG02_Filter_Pages_By_Title'))`

### PG04 — Compose Page Decision
Ref: `PG04 Compose Page Decision`
```
if(greater(outputs('PG03_Compose_Page_Match_Count'), 0), 'PAGE_EXISTS', 'PAGE_NOT_FOUND')
```

### PG05 — Condition Page Exists
11. Add a **Condition**. Name: `PG05 Condition Page Exists`
12. Left (Expression): `outputs('PG04_Compose_Page_Decision')` · is equal to · Right: `PAGE_EXISTS`

#### True branch — Append

**PG06 — Compose Existing Page Id**
Expression: `if(greater(length(body('PG02_Filter_Pages_By_Title')), 0), first(body('PG02_Filter_Pages_By_Title'))?['id'], '')`

**PG07 — Update Page Content (OneNote connector)**
- Notebook Key: `Meeting Notes|$|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Meeting Notes`
- Section ID (Expression): `outputs('PG00_Compose_Target_Section_Pages_Url')`
- Page ID (Expression): `outputs('PG06_Compose_Existing_Page_Id')`
- Target: `body` · Action: `append`
- Content (Expression): `outputs('NZ04_Compose_Update_Html_Fragment')`

**PG08 — Compose Page Action** · Value tab (literal): `UpdatedAppend`

**PG09 — Compose Created Page Link (Append)**
Expression:
```
coalesce(outputs('EM04_Compose_Existing_Page_Web_Url'), '')
```

**PG10 — Compose Created Page Self Url (Append)**
Expression:
```
coalesce(outputs('EM03_Compose_Existing_Page_Self_Url'), '')
```

#### False branch — Create

**PG11 — Create Page In Section (OneNote connector)**
- Notebook Key: `Meeting Notes|$|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Meeting Notes`
- Section ID (Expression): `outputs('PG00_Compose_Target_Section_Pages_Url')`
- Page Content (Expression): `outputs('NZ00d_Compose_PageHtml')`

**PG12 — Compose Page Action** · Value tab (literal): `Created`

**PG13 — Compose Created Page Link (Create)**
Expression:
```
coalesce(outputs('PG11_Create_Page_In_Section')?['body']?['links']?['oneNoteWebUrl']?['href'], '')
```

**PG14 — Compose Created Page Self Url (Create)**
Expression:
```
coalesce(outputs('PG11_Create_Page_In_Section')?['body']?['self'], '')
```

### PG15–PG17 — Coalesce page outputs (after PG05, still inside Scope_PageResolve)

| Ref | Name | Expression |
|---|---|---|
| PG15 | `PG15 Compose Page Action Final` | `coalesce(outputs('PG08_Compose_Page_Action'), outputs('PG12_Compose_Page_Action'), '')` |
| PG16 | `PG16 Compose Page Link Final` | `coalesce(outputs('PG09_Compose_Created_Page_Link'), outputs('PG13_Compose_Created_Page_Link'), '')` |
| PG17 | `PG17 Compose Page Self Url Final` | `coalesce(outputs('PG10_Compose_Created_Page_Self_Url'), outputs('PG14_Compose_Created_Page_Self_Url'), '')` |

Peek Code all PG actions.

---

## Step 9 — Scope_WriteBack (WB)

1. Add a **Scope** after `Scope_PageResolve`. Name: `Scope_WriteBack`
2. runAfter `Scope_PageResolve`: `Succeeded`.

### WB00 — Compose Target Row Id
Expression:
```
coalesce(outputs('NM10_Compose_New_Row_Id'), outputs('EM05_Compose_Existing_Row_Id'), '')
```

### WB01 — Condition Is New Page
3. Add a **Condition**. Name: `WB01 Condition Is New Page`
4. Left (Expression): `outputs('PG15_Compose_Page_Action_Final')` · is equal to · Right: `Created`

#### True branch only — HTTP MERGE

**WB02 — HTTP Update SP Page Urls** (HTTP / Office 365 Outlook connector — same as one-off child)
- Method: `PATCH`
- URI:
```
concat('https://jsainsbury.sharepoint.com/sites/coplt/_api/lists(''186b3c9f-e758-4e85-83d5-685946614a0a'')/items(', outputs('WB00_Compose_Target_Row_Id'), ')')
```
- Headers: `Content-Type` = `application/json;odata=verbose` · `IF-MATCH` = `*` · `X-HTTP-Method` = `MERGE`
- Body:
```
concat('{"__metadata":{"type":"SP.Data.RecurringMeetingSectionMapListItem"},"PageSelfUrl":"', outputs('PG17_Compose_Page_Self_Url_Final'), '","PageWebUrl":"', outputs('PG16_Compose_Page_Link_Final'), '"}')
```

### WB03 — Compose Mapping Write Succeeded (after WB01, outside both branches)
Ref: `WB03 Compose Mapping Write Succeeded`
Expression:
```
coalesce(outputs('NM11_Compose_Mapping_Write_Succeeded'), 'true')
```
(Existing-mapping path has no NM11, so coalesce defaults to `'true'` — matching v1 behaviour.)

---

## Step 10 — Scope_Status (ST)

1. Add a **Scope** after `Scope_WriteBack`. Name: `Scope_Status`

### ST01 — Compose Resolver Result Final
Ref: `ST01 Compose Resolver Result Final`
Expression:
```
coalesce(outputs('NM08_Compose_Resolver_Result_Final'), 'ExistingMapping')
```
(New-mapping path sets NM08; existing-mapping path has none, so coalesce to `ExistingMapping`.)

### ST02 — Compose Out Status
Ref: `ST02 Compose Out Status`

The 7-clause OutStatus precedence from `recurring/scope-map.md`:
```
if(equals(outputs('EM01_Compose_Is_Stale_Row'), true), 'STALE_MAPPING', if(and(contains(createArray('Created','Updated','UpdatedAppend'), outputs('PG15_Compose_Page_Action_Final')), equals(outputs('WB03_Compose_Mapping_Write_Succeeded'), 'true')), 'SUCCESS', if(and(contains(createArray('Created','Updated','UpdatedAppend'), outputs('PG15_Compose_Page_Action_Final')), equals(outputs('WB03_Compose_Mapping_Write_Succeeded'), 'false')), 'PARTIAL_SUCCESS', if(empty(outputs('ST01_Compose_Resolver_Result_Final')), 'RECURRING_SETUP_REQUIRED', if(empty(outputs('PG00_Compose_Target_Section_Pages_Url')), 'SETUP_SECTION_NOT_FOUND', if(greater(outputs('NM03_Compose_Section_Match_Count'), 1), 'SETUP_SECTION_AMBIGUOUS', 'ERROR'))))))
```

⚠ **Note:** `EM01_Compose_Is_Stale_Row` is only populated on the existing-mapping path. On the new-mapping path it is null. `equals(null, true)` evaluates to false in WDL, so the stale clause is safely skipped on the new path. Verify this in run history on first test.

### ST03 — Compose Agent Response Summary
Ref: `ST03 Compose Agent Response Summary`
```
if(equals(outputs('PG15_Compose_Page_Action_Final'), 'Created'), concat('Created a new OneNote meeting page for "', outputs('NZ00a_Compose_MeetingTitle'), '".'), if(equals(outputs('PG15_Compose_Page_Action_Final'), 'UpdatedAppend'), concat('Updated the existing OneNote meeting page for "', outputs('NZ00a_Compose_MeetingTitle'), '" by appending a safe automated update block.'), if(equals(outputs('PG15_Compose_Page_Action_Final'), 'ExistsNoCreate'), concat('An existing OneNote meeting page was found for "', outputs('NZ00a_Compose_MeetingTitle'), '" and is ready for update.'), concat('Processed OneNote meeting page request for "', outputs('NZ00a_Compose_MeetingTitle'), '".'))))
```

Peek Code all ST actions.

---

## Step 11 — Scope_Respond (RP)

1. Add a **Scope** after `Scope_Status`. Name: `Scope_Respond`

### RP01–RP16 — Output Composes

| Ref | Name | Expression |
|---|---|---|
| RP01 | `RP01 Compose outspitemcount` | `int(coalesce(outputs('ML03_Compose_Match_Count'), 0))` |
| RP02 | `RP02 Compose outmatchcount` | `string(outputs('ML03_Compose_Match_Count'))` |
| RP03 | `RP03 Compose outbranchresult` | `string(outputs('ML03_Compose_Match_Count'))` |
| RP04 | `RP04 Compose outonenoteresolverresult` | `outputs('ST01_Compose_Resolver_Result_Final')` |
| RP05 | `RP05 Compose outtargetsectionpagesurl` | `outputs('PG00_Compose_Target_Section_Pages_Url')` |
| RP06 | `RP06 Compose outcreatedpagelink` | `outputs('PG16_Compose_Page_Link_Final')` |
| RP07 | `RP07 Compose outcreatedpageselfurl` | `outputs('PG17_Compose_Page_Self_Url_Final')` |
| RP08 | `RP08 Compose outfinaltargetsectionpagesurl` | `outputs('PG00_Compose_Target_Section_Pages_Url')` |
| RP09 | `RP09 Compose outresolverresult` | `outputs('ST01_Compose_Resolver_Result_Final')` |
| RP10 | `RP10 Compose outexistingpageselfurl` | `coalesce(outputs('EM03_Compose_Existing_Page_Self_Url'), '')` |
| RP11 | `RP11 Compose outpagedecision` | `outputs('PG04_Compose_Page_Decision')` |
| RP12 | `RP12 Compose outpageroute` | `string(equals(outputs('PG04_Compose_Page_Decision'), 'PAGE_EXISTS'))` |
| RP13 | `RP13 Compose outpageaction` | `outputs('PG15_Compose_Page_Action_Final')` |
| RP14 | `RP14 Compose outupdatehtmlfragment` | `outputs('NZ04_Compose_Update_Html_Fragment')` |
| RP15 | `RP15 Compose outagentresponsesummary` | `outputs('ST03_Compose_Agent_Response_Summary')` |
| RP16 | `RP16 Compose outstatus` | `outputs('ST02_Compose_Out_Status')` |

### RP_Respond — Respond to a PowerApp or Flow

2. Add **"Respond to a PowerApp or Flow"**. Name: `RP_Respond`
3. Add all 16 output fields (same keys and titles as the one-off child, bound to RP01–RP16).
4. Peek Code the Respond. Confirm all 16 keys.

---

## Step 12 — Health check, publish, GitHub push

1. Peek Code `Scope_FlowB_Recurring` (outer Scope).
2. Flow Checker — 0 errors.
3. Save draft. Close, wait 10s, reopen, wait 20s.
4. Flow Checker again.
5. Peek Code outer Scope again — cross-reference.
6. Publish.
7. Set connectors in Run-only-users: SharePoint + OneNote — "Use this connection (david.croxson@sainsburys.co.uk)".
8. Push outer Scope Peek Code to `flows/flow-b-v2/scope-peek-codes/scope-recurring.md`.
9. Push known-good values to `flows/flow-b-v2/recurring/known-good-values.md`.

---

## After publishing — wire into the parent

1. Open `PA - Meeting Capture - B - Router - v2`.
2. In the False branch of RT02, add **Run a Child Flow** — name `RT03a Run Recurring Child` — select this flow from the picker.
3. Map inputs per parent→child mapping in `parent/trigger-contract.md`.
4. Set `Scope_Relay` runAfter `Scope_Router` to `Succeeded AND Failed`.
5. Save, Flow Checker, publish, push updated scope-parent.md.
