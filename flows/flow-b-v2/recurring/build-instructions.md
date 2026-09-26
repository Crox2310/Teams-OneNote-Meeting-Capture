# Flow B v2 Recurring Child — Build Instructions

**Flow name:** `PA - Meeting Capture - B - Recurring - v2`
**Build methodology:** Compose-based state passing. No SetVariable. No variables.
**Session-start check:** open flow, wait 20s, Flow Checker, Peek Code `Scope_FlowB_Recurring`.

---

## Prerequisites — BEFORE starting this build

Two inputs are needed from live Flow B v1 that are not in the known-good reference:

1. **Full v1 recurring-branch Peek Code** — specifically the `Apply_to_each_Existing_Section` container and the `Filter_Existing_Section_By_Name` → section-create branch nesting and runAfter chain. Values are in the known-good reference; the *structure* (what runs inside vs outside the loop, what is parallel vs sequential) is not. Pull this before building `Scope_ExistingMapping` and `Scope_NewMapping`.
2. **`Compose_AgentResponseSummary` expression** — not in the known-good reference. Pull from live v1. If it is user-facing (shown to the user in the agent's confirmation message), reproduce it faithfully; if it is diagnostic only, a sensible replacement is acceptable.

Until these are available, build Steps 1–4 (trigger, outer scope, Normalize, MappingLookup) and stop — those are fully specified.

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
3. Description: `Re-expose trigger inputs as named Composes, then derive section name, page title, HTML fragment`

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

Expression — this is the meeting title sanitised for use as a section display name (used by NZ02 as input):
```
coalesce(outputs('NZ00a_Compose_MeetingTitle'), '')
```

### NZ02 — Compose Safe Section Name

Ref: `NZ02 Compose Safe Section Name`

Expression (verbatim from v1 known-good, remapped to NZ01 output):
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

---

## Step 4 — Scope_MappingLookup (ML)

1. Add a **Scope** after `Scope_Normalize`.
2. Name: `Scope_MappingLookup`
3. Description: `Look up existing mapping row for this SeriesMasterId + OccurrenceDate`

### ML01 — Get items (SharePoint connector)

4. Add a **SharePoint — Get items** action.
5. Name: `ML01 Get items`
6. Site Address: `https://jsainsbury.sharepoint.com/sites/coplt`
7. List Name: `RecurringMeetingSectionMap` (GUID `186b3c9f-e758-4e85-83d5-685946614a0a`)
8. Top Count (Advanced): `500`
9. Peek Code. Confirm dataset, table GUID, `$top: 500`.

### ML02 — Filter Existing Mapping (Filter Array)

10. Add a **Filter Array**.
11. Name: `ML02 Filter Existing Mapping`
12. From (Expression): `body('ML01_Get_items')?['value']`
13. Condition:
 - Left (Expression): `item()?['SeriesMasterId']`
 - Operator: `is equal to`
 - Right (Expression): `outputs('NZ00b_Compose_SeriesMasterId')`
14. Add a second condition (And):
 - Left (Expression): `item()?['OccurrenceDate']`
 - Operator: `is equal to`
 - Right (Expression): `outputs('NZ00c_Compose_OccurrenceDate')`
15. Peek Code.

### ML03 — Compose Match Count

16. Add a **Compose**.
17. Name: `ML03 Compose Match Count`
18. Expression: `length(body('ML02_Filter_Existing_Mapping'))`

### ML04 — Compose Mapping Exists

19. Add a **Compose**.
20. Name: `ML04 Compose Mapping Exists`
21. Expression: `greater(outputs('ML03_Compose_Match_Count'), 0)`

---

## ⏸ STOP HERE — build continues once v1 Peek Code is available

The steps below are partially specified. Before building `Scope_ExistingMapping`, `Scope_NewMapping`, and `Scope_PageResolve`, paste the following to the AI assistant in a new session:

1. The Peek Code of `Apply_to_each_Existing_Section` from live v1 (already have the guard — also need `Filter_Existing_Section_By_Name` and the section-create/use branch that precedes it in the recurring new-mapping path).
2. The expression from live v1 `Compose_AgentResponseSummary`.

The AI will then produce the remaining Steps 5–12 at the same level of detail as Steps 1–4.

---

## Steps 5–12 — Partial specification (for reference; complete at next session)

These steps are structurally defined in `flows/flow-b-v2/recurring/scope-map.md`. The action names, Scope names, branch selectors, and OutStatus precedence are all locked. What is missing is the internal action sequence for the existing-section loop (now a Compose pattern, not a foreach+SetVariable) and the `Compose_AgentResponseSummary` expression. Once the v1 Peek Code is available, these steps will be filled in at the same level as Steps 1–4 above.

### Scope_ExistingMapping (EM) — true branch of ML mapping condition
- `EM01 Compose Is Stale Row` — `empty(coalesce(first(body('ML02_Filter_Existing_Mapping'))?['SectionPagesUrl'], ''))`
- `EM02 Compose Existing Section Pages Url` — `coalesce(first(body('ML02_Filter_Existing_Mapping'))?['SectionPagesUrl'], '')`
- `EM03 Compose Existing Page Self Url` — `coalesce(first(body('ML02_Filter_Existing_Mapping'))?['PageSelfUrl'], '')`
- `EM04 Compose Existing Page Web Url` — `coalesce(first(body('ML02_Filter_Existing_Mapping'))?['PageWebUrl'], '')`
- `EM05 Compose Existing Row Id` — `string(first(body('ML02_Filter_Existing_Mapping'))?['ID'])`
- Stale branch selector 1a — `equals(outputs('EM01_Compose_Is_Stale_Row'), true)` → skip to Status; else → PageResolve

### Scope_NewMapping (NM) — false branch
- `NM01 Get Sections` (OneNote connector — GetSectionsInNotebook)
 - notebookKey: `Meeting Notes|$|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Meeting Notes`
- `NM02 Filter Section By Name` (Filter Array) — `item()?['name']` equals `outputs('NZ02_Compose_Safe_Section_Name')`
- `NM03 Compose Section Match Count` — `length(body('NM02_Filter_Section_By_Name'))`
- Branch selector 2 on count: 0 → create, 1 → use existing, >1 → SETUP_SECTION_AMBIGUOUS
 - Create path: `NM04 Create Section` (OneNote CreateSectionInNotebook) — name = `NZ02_Compose_Safe_Section_Name`; then `NM05 Compose Target Section Pages Url` = `outputs('NM04_Create_Section')?['body']?['pagesUrl']`
 - Use-existing path: `NM05 Compose Target Section Pages Url` = `first(body('NM02_Filter_Section_By_Name'))?['pagesUrl']`
- `NM06 Create Mapping Item Recurring` (SharePoint Create item) — all fields per output-contract (SeriesMasterId/MeetingTitle/OccurrenceDate/JoinUrl/EndTime/SectionPagesUrl/Status)
- `NM07 Compose New Row Id` — `string(outputs('NM06...')?['body/ID'])`
- `NM08 Compose Mapping Write Succeeded` — `if(equals(outputs('NM06...')?['statusCode'], 201), 'true', 'false')`

### Scope_PageResolve (PG)
Identical pattern to one-off child Steps 8 (PG01–PG17) except:
- PG01 sectionId = `coalesce(outputs('EM02_Compose_Existing_Section_Pages_Url'), outputs('NM05_Compose_Target_Section_Pages_Url'), '')`
- Append path PG09/PG10 read from EM03/EM04 (recurring equivalents)

### Scope_WriteBack (WB)
Identical pattern to one-off Steps 9 except:
- `WB00 Compose Target Row Id` = `coalesce(outputs('NM07_Compose_New_Row_Id'), outputs('EM05_Compose_Existing_Row_Id'), '')`
- `WB02 Compose Mapping Write Succeeded` reads `NM08`

### Scope_Status (ST)
- `ST01 Compose Out Status` — the 7-clause if nest from `recurring/scope-map.md` OutStatus precedence section, reading Composes throughout
- `ST02 Compose Agent Response Summary` — v1 expression remapped to Compose outputs

### Scope_Respond (RP)
Identical shape to one-off child Step 11 except:
- `RP04 outonenoteresolverresult` = `coalesce(outputs('EM_resolver_compose'), outputs('NM_resolver_compose'), '')`
- `RP05 outtargetsectionpagesurl` = `coalesce(outputs('EM02...'), outputs('NM05...'), '')`
- `RP08 outfinaltargetsectionpagesurl` same as RP05
- `RP09 outresolverresult` = same as RP04
- `RP10 outexistingpageselfurl` = `coalesce(outputs('EM03...'), '')`
