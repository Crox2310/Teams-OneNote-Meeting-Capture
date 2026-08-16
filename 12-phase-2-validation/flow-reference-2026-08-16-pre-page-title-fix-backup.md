# Full Flow Peek Code Backup — 16 August 2026, pre-page-title-fix

Captured immediately before starting the permanent page-title fix (Step 1 of the post-Bug-9 cleanup work). This is the complete, known-good, published state of "PA - Resolve OneNote Meeting Section - v2 Clean Build" at this point — Bug 9 confirmed closed via workaround (see `handover-2026-08-16-bug9-closed-workaround-confirmed.md`), sectionId fix in place, `Compose_RealExistingPageId` using the "section's first page" workaround.

**Use this file to restore or diff against if corruption strikes during the page-title fix work.**

---

## Trigger

```json
{
  "type": "Request",
  "kind": "Skills",
  "inputs": {
    "schema": {
      "type": "object",
      "properties": {
        "text_1": { "description": "Please enter your input", "title": "MeetingTitle", "type": "string", "x-ms-content-hint": "TEXT", "x-ms-dynamically-added": true },
        "text_2": { "description": "Please enter your input", "title": "SeriesMasterId", "type": "string", "x-ms-content-hint": "TEXT", "x-ms-dynamically-added": true },
        "text_3": { "description": "Please enter your input", "title": "PageHtml", "type": "string", "x-ms-content-hint": "TEXT", "x-ms-dynamically-added": true },
        "text_4": { "description": "Please enter your input", "title": "MeetingId", "type": "string", "x-ms-content-hint": "TEXT", "x-ms-dynamically-added": true },
        "text": { "description": "Please enter your input", "title": "IsRecurring", "type": "string", "x-ms-content-hint": "TEXT", "x-ms-dynamically-added": true }
      },
      "required": ["text_1", "text_2", "text_3", "text_4", "text"]
    }
  },
  "metadata": { "flowSystemMetadata": { "flowKind": "Stateless" } }
}
```

## Get items

```json
{
  "type": "OpenApiConnection",
  "inputs": {
    "parameters": {
      "dataset": "https://jsainsbury.sharepoint.com/sites/coplt",
      "table": "186b3c9f-e758-4e85-83d5-685946614a0a"
    },
    "host": { "apiId": "/providers/Microsoft.PowerApps/apis/shared_sharepointonline", "connection": "shared_sharepointonline", "operationId": "GetItems" }
  },
  "runAfter": {}
}
```

## InitializeVariable actions (in order)

```json
{ "type": "InitializeVariable", "inputs": { "variables": [{ "name": "varFinalExistingPageSelfUrl", "type": "string" }] }, "runAfter": { "Get_items": ["Succeeded"] } }
```
```json
{ "type": "InitializeVariable", "inputs": { "variables": [{ "name": "varFinalPageDecision", "type": "string" }] }, "runAfter": { "varFinalExistingPageSelfUrl": ["SUCCEEDED"] } }
```
```json
{ "type": "InitializeVariable", "inputs": { "variables": [{ "name": "varFinalMatchCount", "type": "string" }] }, "runAfter": { "varFinalPageDecision": ["SUCCEEDED"] } }
```
```json
{ "type": "InitializeVariable", "inputs": { "variables": [{ "name": "varOutStatus", "type": "string", "value": "OK" }] }, "runAfter": { "varFinalMatchCount": ["SUCCEEDED"] } }
```
```json
{ "type": "InitializeVariable", "inputs": { "variables": [{ "name": "varOutputPageLink", "type": "string" }] }, "runAfter": { "varOutStatus": ["SUCCEEDED"] } }
```
```json
{ "type": "InitializeVariable", "inputs": { "variables": [{ "name": "varOutputPageSelfUrl", "type": "string" }] }, "runAfter": { "varOutputPageLink": ["SUCCEEDED"] } }
```
```json
{ "type": "InitializeVariable", "inputs": { "variables": [{ "name": "varTargetSectionPagesUrl", "type": "string" }] }, "runAfter": { "varOutputPageSelfUrl": ["SUCCEEDED"] } }
```
```json
{ "type": "InitializeVariable", "inputs": { "variables": [{ "name": "varOneNoteResolverResult", "type": "string" }] }, "runAfter": { "varTargetSectionPagesUrl": ["SUCCEEDED"] } }
```
```json
{ "type": "InitializeVariable", "inputs": { "variables": [{ "name": "varPageAction", "type": "string" }] }, "runAfter": { "varOneNoteResolverResult": ["SUCCEEDED"] } }
```

## Condition IsRecurring (full, both branches)

```json
{
  "type": "If",
  "expression": { "and": [{ "equals": ["@equals(toLower(string(triggerBody()?['text'])), 'true')", true] }] },
  "actions": {
    "Compose_Input_SeriesMasterId": { "type": "Compose", "inputs": "@triggerBody()?['text_2']" },
    "Compose_Input_MeetingTitle": { "type": "Compose", "inputs": "@triggerBody()?['text_1']", "runAfter": { "Compose_Input_SeriesMasterId": ["Succeeded"] } },
    "Filter_Existing_Mapping": { "type": "Query", "inputs": { "from": "@body('Get_items')?['value']", "where": "@equals(item()?['SeriesMasterId'],triggerBody()?['text_2'])" }, "runAfter": { "Compose_Input_MeetingTitle": ["Succeeded"] } },
    "Compose_ExistingPageSelfUrl": { "type": "Compose", "inputs": "@if(\n  greater(length(body('Filter_Existing_Mapping')), 0),\n  first(body('Filter_Existing_Mapping'))?['PageSelfUrl'],\n  ''\n)", "runAfter": { "Filter_Existing_Mapping": ["Succeeded"] } },
    "Compose_PageDecision": { "type": "Compose", "inputs": "@if(\n  not(empty(outputs('Compose_ExistingPageSelfUrl'))),\n  'PAGE_EXISTS',\n  'PAGE_NOT_FOUND'\n)", "runAfter": { "Compose_ExistingPageSelfUrl": ["Succeeded"] } },
    "Compose_Match_Count": { "type": "Compose", "inputs": "@length(body('Filter_Existing_Mapping'))", "runAfter": { "Compose_PageDecision": ["Succeeded"] } },
    "varFinalExistingPageSelfUrl_1": { "type": "SetVariable", "inputs": { "name": "varFinalExistingPageSelfUrl", "value": "@outputs('Compose_ExistingPageSelfUrl')" }, "runAfter": { "Compose_Match_Count": ["Succeeded"] } },
    "varFinalPageDecision_1": { "type": "SetVariable", "inputs": { "name": "varFinalPageDecision", "value": "@outputs('Compose_PageDecision')" }, "runAfter": { "varFinalExistingPageSelfUrl_1": ["Succeeded"] } },
    "varFinalMatchCount_1": { "type": "SetVariable", "inputs": { "name": "varFinalMatchCount", "value": "@string(outputs('Compose_Match_Count'))" }, "runAfter": { "varFinalPageDecision_1": ["Succeeded"] } }
  },
  "else": {
    "actions": {
      "FB-F01_—_Compose_Input_MeetingTitle_(one-off)": { "type": "Compose", "inputs": "@if(empty(trim(coalesce(triggerBody()?['text_1'], ''))), 'Mtg - Untitled Meeting', concat('Mtg - ', substring(replace(replace(replace(replace(replace(replace(replace(replace(triggerBody()?['text_1'], '/', '-'), ':', '-'), '&', 'and'), '?', ''), '*', ''), '<', ''), '>', ''), '\"', ''), 0, min(43, length(replace(replace(replace(replace(replace(replace(replace(replace(triggerBody()?['text_1'], '/', '-'), ':', '-'), '&', 'and'), '?', ''), '*', ''), '<', ''), '>', ''), '\"', ''))))))" },
      "Get_Sections_OneOff": { "type": "OpenApiConnection", "inputs": { "parameters": { "notebookKey": "Meeting Notes|$|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Meeting Notes" }, "host": { "apiId": "/providers/Microsoft.PowerApps/apis/shared_onenote", "connection": "shared_onenote-1", "operationId": "GetSectionsInNotebook" } }, "runAfter": { "FB-F01_—_Compose_Input_MeetingTitle_(one-off)": ["SUCCEEDED"] } },
      "Filter_OneNote_Section_OneOff": { "type": "Query", "inputs": { "from": "@outputs('Get_Sections_OneOff')?['body/value']", "where": "@equals(item()?['name'], outputs('FB-F01_—_Compose_Input_MeetingTitle_(one-off)'))" }, "runAfter": { "Get_Sections_OneOff": ["SUCCEEDED"] } },
      "Compose_Section_Match_Count_OneOff": { "type": "Compose", "inputs": "@length(body('Filter_OneNote_Section_OneOff'))", "runAfter": { "Filter_OneNote_Section_OneOff": ["SUCCEEDED"] } },
      "Condition_Section_Exists_OneOff": {
        "type": "If",
        "expression": { "and": [{ "equals": ["@greater(outputs('Compose_Section_Match_Count_OneOff'), 0)", "@true"] }] },
        "actions": {
          "For_each_1": { "type": "foreach", "foreach": "@body('Filter_OneNote_Section_OneOff')", "actions": {
            "Set_varTargetSectionPagesUrl_OneOff_Exists": { "type": "SetVariable", "inputs": { "name": "varTargetSectionPagesUrl", "value": "@items('For_each_1')?['pagesUrl']" } },
            "Set_varOneNoteResolverResult_Exists_OneOff": { "type": "SetVariable", "inputs": { "name": "varOneNoteResolverResult", "value": "ExistingSection" }, "runAfter": { "Set_varTargetSectionPagesUrl_OneOff_Exists": ["SUCCEEDED"] } }
          } }
        },
        "else": { "actions": {
          "Create_Section_OneOff": { "type": "OpenApiConnection", "inputs": { "parameters": { "body/name": "@outputs('FB-F01_—_Compose_Input_MeetingTitle_(one-off)')", "notebookKey": "Meeting Notes|$|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Meeting Notes" }, "host": { "apiId": "/providers/Microsoft.PowerApps/apis/shared_onenote", "connection": "shared_onenote-1", "operationId": "CreateSectionInNotebook" } } },
          "Set_varTargetSectionPagesUrl_OneOff_Created": { "type": "SetVariable", "inputs": { "name": "varTargetSectionPagesUrl", "value": "@outputs('Create_Section_OneOff')?['body']?['pagesUrl']" }, "runAfter": { "Create_Section_OneOff": ["SUCCEEDED"] } },
          "Set_varOneNoteResolverResult_Created_OneOff": { "type": "SetVariable", "inputs": { "name": "varOneNoteResolverResult", "value": "CreatedSection" }, "runAfter": { "Set_varTargetSectionPagesUrl_OneOff_Created": ["SUCCEEDED"] } }
        } },
        "runAfter": { "Compose_Section_Match_Count_OneOff": ["SUCCEEDED"] }
      },
      "OF01_—_Filter_Existing_Mapping_OneOff": { "type": "Query", "inputs": { "from": "@body('Get_items')?['value']", "where": "@equals(item()?['MeetingId'],triggerBody()?['text_4'])" }, "runAfter": { "Condition_Section_Exists_OneOff": ["SUCCEEDED"] } },
      "OF02_—_Compose_ExistingPageSelfUrl_OneOff": { "type": "Compose", "inputs": "@if(greater(length(body('OF01_—_Filter_Existing_Mapping_OneOff')), 0), first(body('OF01_—_Filter_Existing_Mapping_OneOff'))?['PageSelfUrl'], '')", "runAfter": { "OF01_—_Filter_Existing_Mapping_OneOff": ["SUCCEEDED"] } },
      "OF03_—_Compose_PageDecision_OneOff": { "type": "Compose", "inputs": "@if(not(empty(outputs('OF02_—_Compose_ExistingPageSelfUrl_OneOff'))), 'PAGE_EXISTS', 'PAGE_NOT_FOUND')", "runAfter": { "OF02_—_Compose_ExistingPageSelfUrl_OneOff": ["SUCCEEDED"] } },
      "OF04_—_Compose_Match_Count_OneOff": { "type": "Compose", "inputs": "@length(body('OF01_—_Filter_Existing_Mapping_OneOff'))", "runAfter": { "OF03_—_Compose_PageDecision_OneOff": ["SUCCEEDED"] } },
      "OF05a_—_Set_varFinalExistingPageSelfUrl_(OneOff)": { "type": "SetVariable", "inputs": { "name": "varFinalExistingPageSelfUrl", "value": "@outputs('OF02_—_Compose_ExistingPageSelfUrl_OneOff')" }, "runAfter": { "OF04_—_Compose_Match_Count_OneOff": ["SUCCEEDED"] } },
      "OF05b_—_Set_varFinalPageDecision_(OneOff)": { "type": "SetVariable", "inputs": { "name": "varFinalPageDecision", "value": "@outputs('OF03_—_Compose_PageDecision_OneOff')" }, "runAfter": { "OF05a_—_Set_varFinalExistingPageSelfUrl_(OneOff)": ["SUCCEEDED"] } },
      "OF05c_—_Set_varFinalMatchCount_(OneOff)": { "type": "SetVariable", "inputs": { "name": "varFinalMatchCount", "value": "@string(outputs('OF04_—_Compose_Match_Count_OneOff'))" }, "runAfter": { "OF05b_—_Set_varFinalPageDecision_(OneOff)": ["SUCCEEDED"] } }
    }
  },
  "runAfter": { "varPageAction": ["Succeeded"] }
}
```

## Condition Mapping Exists (full, both branches)

```json
{
  "type": "If",
  "expression": { "and": [{ "equals": ["@greater(int(if(empty(variables('varFinalMatchCount')), '0', variables('varFinalMatchCount'))), 0)", "@true"] }] },
  "actions": {
    "Compose_Branch_Result": { "type": "Compose", "inputs": "EXISTS", "runAfter": { "Compose_PageRoute_Exists": ["Succeeded"] } },
    "Set_varOneNoteResolverResult_ExistingMapping": { "type": "SetVariable", "inputs": { "name": "varOneNoteResolverResult", "value": "ExistingMapping" }, "runAfter": { "Condition_Recurring_TargetSection": ["SUCCEEDED"] } },
    "Compose_PageRoute_Exists": { "type": "Compose", "inputs": "PAGE_EXISTS_ROUTE" },
    "Condition_Recurring_TargetSection": {
      "type": "If",
      "expression": { "and": [{ "equals": ["@equals(toLower(string(triggerBody()?['text'])), 'true')", true] }] },
      "actions": { "Set_varTargetSectionPagesUrl_ExistingMapping": { "type": "SetVariable", "inputs": { "name": "varTargetSectionPagesUrl", "value": "@first(body('Filter_Existing_Mapping'))?['SectionPagesUrl']" } } },
      "else": { "actions": {} },
      "runAfter": { "Compose_Branch_Result": ["Succeeded"] }
    }
  },
  "else": {
    "actions": {
      "Compose_Branch_Result_NoMatch": { "type": "Compose", "inputs": "CREATE_REQUIRED", "runAfter": { "Compose_PageRoute_CreateRequired": ["Succeeded"] } },
      "Condition_Should_Write_Mapping": {
        "type": "If",
        "expression": { "and": [{ "equals": ["@equals(toLower(string(triggerBody()?['text'])), 'true')", "@true"] }] },
        "actions": { "Send_an_HTTP_request_to_SharePoint": { "type": "OpenApiConnection", "inputs": { "parameters": { "dataset": "https://jsainsbury.sharepoint.com/sites/coplt", "parameters/method": "POST", "parameters/uri": "_api/web/lists/GetByTitle('RecurringMeetingSectionMap')/items", "parameters/headers": { "Accept": "application/json;odata=nometadata", "Content-Type": "application/json;odata=nometadata" }, "parameters/body": "{\n  \"Title\": \"Mapping\",\n  \"SeriesMasterId\": \"@{outputs('Compose_Input_SeriesMasterId')}\",\n  \"MeetingTitle\": \"@{outputs('Compose_Input_MeetingTitle')}\",\n  \"SectionPagesUrl\": \"@{variables('varTargetSectionPagesUrl')}\",\n  \"Status\": \"Active\"\n}" }, "host": { "apiId": "/providers/Microsoft.PowerApps/apis/shared_sharepointonline", "connection": "shared_sharepointonline", "operationId": "HttpRequest" } } } },
        "else": { "actions": {} },
        "runAfter": { "Condition_Section_Exists_Recurring": ["SUCCEEDED"] }
      },
      "Compose_PageRoute_CreateRequired": { "type": "Compose", "inputs": "PAGE_NOT_FOUND_ROUTE", "runAfter": { "Compose_IgnoreSeriesMasterId": ["Succeeded"] } },
      "Compose_IgnoreSeriesMasterId": { "type": "Compose", "inputs": "''" },
      "Compose_SectionDisplayName": { "type": "Compose", "inputs": "@triggerBody()?['text_1']", "runAfter": { "Compose_Branch_Result_NoMatch": ["Succeeded"] } },
      "Compose_SafeSectionName": { "type": "Compose", "inputs": "@if(empty(trim(coalesce(outputs('Compose_SectionDisplayName'), ''))), 'Mtg - Untitled Meeting', concat('Mtg - ', substring(replace(replace(replace(replace(replace(replace(replace(replace(outputs('Compose_SectionDisplayName'), '/', '-'), ':', '-'), '&', 'and'), '?', ''), '*', ''), '<', ''), '>', ''), '\"', ''), 0, min(43, length(replace(replace(replace(replace(replace(replace(replace(replace(outputs('Compose_SectionDisplayName'), '/', '-'), ':', '-'), '&', 'and'), '?', ''), '*', ''), '<', ''), '>', ''), '\"', ''))))))", "runAfter": { "Compose_SectionDisplayName": ["SUCCEEDED"] } },
      "Get_Sections_Recurring": { "type": "OpenApiConnection", "inputs": { "parameters": { "notebookKey": "Meeting Notes|$|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Meeting Notes" }, "host": { "apiId": "/providers/Microsoft.PowerApps/apis/shared_onenote", "connection": "shared_onenote-1", "operationId": "GetSectionsInNotebook" } }, "runAfter": { "Compose_SafeSectionName": ["SUCCEEDED"] } },
      "Filter_OneNote_Section_Recurring": { "type": "Query", "inputs": { "from": "@outputs('Get_Sections_Recurring')?['body/value']", "where": "@equals(item()?['name'],outputs('Compose_SafeSectionName'))" }, "runAfter": { "Get_Sections_Recurring": ["SUCCEEDED"] } },
      "Condition_Section_Exists_Recurring": {
        "type": "If",
        "expression": { "and": [{ "equals": ["@greater(outputs('Compose_Section_Match_Count_Recurring'), 0)", "@true"] }] },
        "actions": { "Apply_to_each": { "type": "Foreach", "foreach": "@body('Filter_OneNote_Section_Recurring')", "actions": {
          "varTargetSectionPagesUrl_1": { "type": "SetVariable", "inputs": { "name": "varTargetSectionPagesUrl", "value": "@items('Apply_to_each')?['pagesUrl']" } },
          "varOneNoteResolverResult_1": { "type": "SetVariable", "inputs": { "name": "varOneNoteResolverResult", "value": "ExistingSection" }, "runAfter": { "varTargetSectionPagesUrl_1": ["SUCCEEDED"] } }
        } } },
        "else": { "actions": {
          "Create_Section_Recurring": { "type": "OpenApiConnection", "inputs": { "parameters": { "body/name": "@outputs('Compose_SafeSectionName')", "notebookKey": "Meeting Notes|$|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Meeting Notes" }, "host": { "apiId": "/providers/Microsoft.PowerApps/apis/shared_onenote", "connection": "shared_onenote-1", "operationId": "CreateSectionInNotebook" } } },
          "varTargetSectionPagesUrl_2": { "type": "SetVariable", "inputs": { "name": "varTargetSectionPagesUrl", "value": "@outputs('Create_Section_Recurring')?['body']?['pagesUrl']" }, "runAfter": { "Create_Section_Recurring": ["SUCCEEDED"] } },
          "varOneNoteResolverResult_2": { "type": "SetVariable", "inputs": { "name": "varOneNoteResolverResult", "value": "CreatedSection" }, "runAfter": { "varTargetSectionPagesUrl_2": ["SUCCEEDED"] } }
        } },
        "runAfter": { "Compose_Section_Match_Count_Recurring": ["SUCCEEDED"] }
      },
      "Compose_Section_Match_Count_Recurring": { "type": "Compose", "inputs": "@length(body('Filter_OneNote_Section_Recurring'))", "runAfter": { "Filter_OneNote_Section_Recurring": ["SUCCEEDED"] } }
    }
  },
  "runAfter": { "Condition_IsRecurring": ["Succeeded"] }
}
```

## Condition Should Create Page (full, both branches — includes Create_OneNote_Page, Create_Page_OneOff, and today's Bug 9 workaround chain)

```json
{
  "type": "If",
  "expression": { "and": [{ "equals": ["@equals(variables('varFinalPageDecision'), 'PAGE_NOT_FOUND')", true] }] },
  "actions": {
    "Create_OneNote_Page": { "type": "OpenApiConnection", "inputs": { "parameters": { "notebookKey": "Meeting Notes|$|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Meeting Notes", "sectionId": "@variables('varTargetSectionPagesUrl')", "pageContent": "<p class=\"editor-paragraph\">@{triggerBody()?['text_3']}</p>" }, "host": { "apiId": "/providers/Microsoft.PowerApps/apis/shared_onenote", "connection": "shared_onenote", "operationId": "CreatePageInSection" } } },
    "Compose_PageSelfUrl_Created": { "type": "Compose", "inputs": "@body('Create_OneNote_Page')?['self']", "runAfter": { "Create_OneNote_Page": ["Succeeded"] } },
    "OF09-Gate_—_Condition_Is_Recurring_(SP_Write)": {
      "type": "If",
      "expression": { "and": [{ "equals": ["@equals(toLower(string(triggerBody()?['text'])), 'true')", true] }] },
      "actions": {
        "HTTP_Update_SP_PageSelfUrl": { "type": "OpenApiConnection", "inputs": { "parameters": { "dataset": "https://jsainsbury.sharepoint.com/sites/coplt", "parameters/method": "POST", "parameters/uri": "_api/web/lists/GetByTitle('RecurringMeetingSectionMap')/items(@{if(greater(length(body('Filter_Existing_Mapping')),0), first(body('Filter_Existing_Mapping'))?['ID'], body('Send_an_HTTP_request_to_SharePoint')?['ID'])})", "parameters/headers": { "Accept": "application/json;odata=nometadata", "Content-Type": "application/json;odata=nometadata", "IF-MATCH": "*", "X-HTTP-Method": "MERGE" }, "parameters/body": "{\n     \"PageSelfUrl\": \"@{outputs('Compose_PageSelfUrl_Created')}\",\n     \"PageWebUrl\": \"@{outputs('Create_OneNote_Page')?['body']?['links']?['oneNoteWebUrl']?['href']}\"\n   }" }, "host": { "apiId": "/providers/Microsoft.PowerApps/apis/shared_sharepointonline", "connection": "shared_sharepointonline", "operationId": "HttpRequest" } } },
        "Set_varPageAction_Created": { "type": "SetVariable", "inputs": { "name": "varPageAction", "value": "Created" }, "runAfter": { "HTTP_Update_SP_PageSelfUrl": ["SUCCEEDED"] } },
        "Set_varOutputPageSelfUrl_Created": { "type": "SetVariable", "inputs": { "name": "varOutputPageSelfUrl", "value": "@outputs('Compose_PageSelfUrl_Created')" }, "runAfter": { "Set_varPageAction_Created": ["SUCCEEDED"] } },
        "Set_varOutputPageLink_Created": { "type": "SetVariable", "inputs": { "name": "varOutputPageLink", "value": "@outputs('Create_OneNote_Page')?['body']?['links']?['oneNoteWebUrl']?['href']" }, "runAfter": { "Set_varOutputPageSelfUrl_Created": ["SUCCEEDED"] } }
      },
      "else": {
        "actions": {
          "OF09b-i_—_Condition_Should_Insert_Mapping_(OneOff)": {
            "type": "If",
            "expression": { "and": [{ "equals": ["@equals(length(body('OF01_—_Filter_Existing_Mapping_OneOff')), 0)", true] }] },
            "actions": { "OF09a_—_Send_an_HTTP_request_to_SharePoint_(OneOff)": { "type": "OpenApiConnection", "inputs": { "parameters": { "dataset": "https://jsainsbury.sharepoint.com/sites/coplt", "parameters/method": "POST", "parameters/uri": "_api/web/lists/GetByTitle('RecurringMeetingSectionMap')/items", "parameters/headers": { "Accept": "application/json;odata=nometadata", "Content-Type": "application/json;odata=nometadata" }, "parameters/body": "{\n       \"Title\": \"Mapping\",\n       \"MeetingId\": \"@{triggerBody()?['text_4']}\",\n       \"MeetingTitle\": \"@{outputs('FB-F01_—_Compose_Input_MeetingTitle_(one-off)')}\",\n       \"SectionPagesUrl\": \"@{variables('varTargetSectionPagesUrl')}\",\n       \"Status\": \"Active\"\n     }" }, "host": { "apiId": "/providers/Microsoft.PowerApps/apis/shared_sharepointonline", "connection": "shared_sharepointonline", "operationId": "HttpRequest" } } } },
            "else": { "actions": {} }
          },
          "OF09b_—_HTTP_Update_SP_PageSelfUrl_(OneOff)": { "type": "OpenApiConnection", "inputs": { "parameters": { "dataset": "https://jsainsbury.sharepoint.com/sites/coplt", "parameters/method": "POST", "parameters/uri": "_api/web/lists/GetByTitle('RecurringMeetingSectionMap')/items(@{if(greater(length(body('OF01_—_Filter_Existing_Mapping_OneOff')),0), first(body('OF01_—_Filter_Existing_Mapping_OneOff'))?['ID'], body('OF09a_—_Send_an_HTTP_request_to_SharePoint_(OneOff)')?['ID'])})", "parameters/headers": { "Accept": "application/json;odata=nometadata", "Content-Type": "application/json;odata=nometadata", "IF-MATCH": "*", "X-HTTP-Method": "MERGE" }, "parameters/body": "{\n     \"PageSelfUrl\": \"@{outputs('Compose_PageSelfUrl_Created')}\",\n     \"PageWebUrl\": \"@{body('Create_OneNote_Page')?['links']?['oneNoteWebUrl']?['href']}\"\n   }" }, "host": { "apiId": "/providers/Microsoft.PowerApps/apis/shared_sharepointonline", "connection": "shared_sharepointonline", "operationId": "HttpRequest" } }, "runAfter": { "OF09b-i_—_Condition_Should_Insert_Mapping_(OneOff)": ["SUCCEEDED"] } },
          "Set_varPageAction_Created_OneOff": { "type": "SetVariable", "inputs": { "name": "varPageAction", "value": "Created" }, "runAfter": { "OF09b_—_HTTP_Update_SP_PageSelfUrl_(OneOff)": ["SUCCEEDED"] } },
          "Set_varOutputPageSelfUrl_Created_OneOff": { "type": "SetVariable", "inputs": { "name": "varOutputPageSelfUrl", "value": "@outputs('Compose_PageSelfUrl_Created')" }, "runAfter": { "Set_varPageAction_Created_OneOff": ["SUCCEEDED"] } },
          "Set_varOutputPageLink_Created_OneOff_Gate": { "type": "SetVariable", "inputs": { "name": "varOutputPageLink", "value": "@outputs('Create_OneNote_Page')?['body']?['links']?['oneNoteWebUrl']?['href']" }, "runAfter": { "Set_varOutputPageSelfUrl_Created_OneOff": ["SUCCEEDED"] } }
        }
      },
      "runAfter": { "Compose_PageSelfUrl_Created": ["Succeeded"] }
    }
  },
  "else": {
    "actions": {
      "Set_varPageAction_ExistsNoCreate": { "type": "SetVariable", "inputs": { "name": "varPageAction", "value": "Updated" } },
      "Set_varOutputPageSelfUrl_Existing": { "type": "SetVariable", "inputs": { "name": "varOutputPageSelfUrl", "value": "@variables('varFinalExistingPageSelfUrl')" }, "runAfter": { "Set_varPageAction_ExistsNoCreate": ["Succeeded"] } },
      "Compose_UpdateHtmlFragment": { "type": "Compose", "inputs": "<hr>\n<h2>Automated update</h2>\n<p><strong>Updated by:</strong> Meeting Capture Agent</p>\n<p><strong>Update note:</strong> Meeting details were refreshed by the automation. Existing human-entered notes were preserved.</p>", "runAfter": { "Set_varOutputPageSelfUrl_Existing": ["Succeeded"] } },
      "Compose_ExistingPageId": { "type": "Compose", "inputs": "@last(split(variables('varOutputPageSelfUrl'), '/'))", "runAfter": { "Compose_UpdateHtmlFragment": ["Succeeded"] } },
      "Condition_Is_Genuine_Existing_Page": {
        "type": "If",
        "expression": { "and": [{ "equals": ["@contains(createArray('ExistingMapping', 'ExistingSection'), variables('varOneNoteResolverResult'))", true] }] },
        "actions": {
          "Get_Sections_Existing_Branch": { "type": "OpenApiConnection", "inputs": { "parameters": { "notebookKey": "Meeting Notes|$|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Meeting Notes" }, "host": { "apiId": "/providers/Microsoft.PowerApps/apis/shared_onenote", "connection": "shared_onenote", "operationId": "GetSectionsInNotebook" } } },
          "Filter_Existing_Section_By_Name": { "type": "Query", "inputs": { "from": "@outputs('Get_Sections_Existing_Branch')?['body/value']", "where": "@equals(item()?['name'],outputs('Compose_SafeSectionName_ExistingBranch'))" }, "runAfter": { "Compose_SafeSectionName_ExistingBranch": ["SUCCEEDED"] } },
          "Compose_SectionDisplayName_ExistingBranch": { "type": "Compose", "inputs": "@triggerBody()?['text_1']", "runAfter": { "Get_Sections_Existing_Branch": ["SUCCEEDED"] } },
          "Compose_SafeSectionName_ExistingBranch": { "type": "Compose", "inputs": "@if(empty(trim(coalesce(outputs('Compose_SectionDisplayName_ExistingBranch'), ''))), 'Mtg - Untitled Meeting', concat('Mtg - ', substring(replace(replace(replace(replace(replace(replace(replace(replace(outputs('Compose_SectionDisplayName_ExistingBranch'), '/', '-'), ':', '-'), '&', 'and'), '?', ''), '*', ''), '<', ''), '>', ''), '\"', ''), 0, min(43, length(replace(replace(replace(replace(replace(replace(replace(replace(outputs('Compose_SectionDisplayName_ExistingBranch'), '/', '-'), ':', '-'), '&', 'and'), '?', ''), '*', ''), '<', ''), '>', ''), '\"', ''))))))", "runAfter": { "Compose_SectionDisplayName_ExistingBranch": ["SUCCEEDED"] } },
          "Apply_to_each_Existing_Section": {
            "type": "Foreach",
            "foreach": "@body('Filter_Existing_Section_By_Name')",
            "actions": {
              "Update_page_content_Existing_Branch": { "type": "OpenApiConnection", "inputs": { "parameters": { "notebookKey": "Meeting Notes|$|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Meeting Notes", "sectionId": "@items('Apply_to_each_Existing_Section')?['pagesUrl']", "pageId": "@outputs('Compose_RealExistingPageId')", "updates": [ { "target": "body", "action": "append", "position": "after", "content": "@outputs('Compose_UpdateHtmlFragment')" } ] }, "host": { "apiId": "/providers/Microsoft.PowerApps/apis/shared_onenote", "connection": "shared_onenote", "operationId": "UpdatePageContent" } }, "runAfter": { "Compose_RealExistingPageId": ["SUCCEEDED"] } },
              "Get_Pages_In_Section_Existing_Branch": { "type": "OpenApiConnection", "inputs": { "parameters": { "notebookKey": "Meeting Notes|$|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Meeting Notes", "sectionId": "@items('Apply_to_each_Existing_Section')?['pagesUrl']" }, "host": { "apiId": "/providers/Microsoft.PowerApps/apis/shared_onenote", "connection": "shared_onenote-1", "operationId": "GetPagesInSection" } } },
              "Compose_MeetingTitleForPageMatch": { "type": "Compose", "inputs": "@triggerBody()?['text_1']", "runAfter": { "Get_Pages_In_Section_Existing_Branch": ["SUCCEEDED"] } },
              "Filter_Pages_By_Title": { "type": "Query", "inputs": { "from": "@outputs('Get_Pages_In_Section_Existing_Branch')?['body/value']", "where": "@equals(item()?['title'],outputs('Compose_MeetingTitleForPageMatch'))" }, "runAfter": { "Compose_MeetingTitleForPageMatch": ["SUCCEEDED"] } },
              "Compose_RealExistingPageId": { "type": "Compose", "inputs": "@if(greater(length(outputs('Get_Pages_In_Section_Existing_Branch')?['body']?['value']), 0), first(outputs('Get_Pages_In_Section_Existing_Branch')?['body']?['value'])?['id'], '')", "runAfter": { "Filter_Pages_By_Title": ["SUCCEEDED"] } }
            },
            "runAfter": { "Filter_Existing_Section_By_Name": ["SUCCEEDED"] }
          },
          "Set_varPageAction_UpdatedAppend": { "type": "SetVariable", "inputs": { "name": "varPageAction", "value": "Updated" }, "runAfter": { "Apply_to_each_Existing_Section": ["SUCCEEDED"] } },
          "Set_varOutputPageLink_Existing": { "type": "SetVariable", "inputs": { "name": "varOutputPageLink", "value": "@variables('varFinalExistingPageSelfUrl')" }, "runAfter": { "Set_varPageAction_UpdatedAppend": ["SUCCEEDED"] } }
        },
        "else": {
          "actions": {
            "Create_Page_OneOff": { "type": "OpenApiConnection", "inputs": { "parameters": { "notebookKey": "Meeting Notes|$|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Meeting Notes", "sectionId": "@variables('varTargetSectionPagesUrl')", "pageContent": "<p class=\"editor-paragraph\">@{triggerBody()?['text_3']}</p>" }, "host": { "apiId": "/providers/Microsoft.PowerApps/apis/shared_onenote", "connection": "shared_onenote-1", "operationId": "CreatePageInSection" } } },
            "Set_varOutputPageLink_Created_OneOff": { "type": "SetVariable", "inputs": { "name": "varOutputPageLink", "value": "@outputs('Create_Page_OneOff')?['body']?['links']?['oneNoteWebUrl']?['href']" }, "runAfter": { "Create_Page_OneOff": ["SUCCEEDED"] } }
          }
        },
        "runAfter": { "Compose_ExistingPageId": ["Succeeded"] }
      }
    }
  },
  "runAfter": { "Condition_Mapping_Exists": ["Succeeded"] }
}
```

## Tail: response summary, item count, status, respond to agent

```json
{ "type": "Compose", "inputs": "@if(\n  equals(variables('varPageAction'), 'Created'),\n  concat('Created a new OneNote meeting page for \"', triggerBody()?['text_1'], '\".'),\n  if(\n    equals(variables('varPageAction'), 'UpdatedAppend'),\n    concat('Updated the existing OneNote meeting page for \"', triggerBody()?['text_1'], '\" by appending a safe automated update block.'),\n    if(\n      equals(variables('varPageAction'), 'ExistsNoCreate'),\n      concat('An existing OneNote meeting page was found for \"', triggerBody()?['text_1'], '\" and is ready for update.'),\n      concat('Processed OneNote meeting page request for \"', triggerBody()?['text_1'], '\".')\n    )\n  )\n)", "runAfter": { "Condition_Should_Create_Page": ["Succeeded"] } }
```
```json
{ "type": "Compose", "inputs": "@length(body('Get_items')?['value'])", "runAfter": { "Compose_AgentResponseSummary": ["Succeeded"] } }
```
```json
{ "type": "SetVariable", "inputs": { "name": "varOutStatus", "value": "OK" }, "runAfter": { "Compose_SP_Item_Count": ["Succeeded"] } }
```
```json
{
  "type": "Response",
  "kind": "Skills",
  "inputs": {
    "schema": { "type": "object", "properties": {
      "outisrecurring": { "title": "OutIsRecurring", "type": "string" }, "outmeetingtitle": { "title": "OutMeetingTitle", "type": "string" }, "outseriesmasterid": { "title": "OutSeriesMasterId", "type": "string" }, "outpagehtml": { "title": "OutPageHtml", "type": "string" }, "outspitemcount": { "title": "OutSPItemCount", "type": "string" }, "outmatchcount": { "title": "OutMatchCount", "type": "string" }, "outbranchresult": { "title": "OutBranchResult", "type": "string" }, "outonenoteresolverresult": { "title": "OutOneNoteResolverResult", "type": "string" }, "outtargetsectionpagesurl": { "title": "OutTargetSectionPagesUrl", "type": "string" }, "outcreatedpagelink": { "title": "OutCreatedPageLink", "type": "string" }, "outcreatedpageselfurl": { "title": "OutCreatedPageSelfUrl", "type": "string" }, "outfinaltargetsectionpagesurl": { "title": "OutFinalTargetSectionPagesUrl", "type": "string" }, "outresolverresult": { "title": "OutResolverResult", "type": "string" }, "outexistingpageselfurl": { "title": "OutExistingPageSelfUrl", "type": "string" }, "outpagedecision": { "title": "OutPageDecision", "type": "string" }, "outpageroute": { "title": "OutPageRoute", "type": "string" }, "outpageaction": { "title": "OutPageAction", "type": "string" }, "outupdatehtmlfragment": { "title": "OutUpdateHtmlFragment", "type": "string" }, "outagentresponsesummary": { "title": "OutAgentResponseSummary", "type": "string" }, "outstatus": { "title": "OutStatus", "type": "string" }
    }, "additionalProperties": {} },
    "statusCode": 200,
    "body": {
      "outisrecurring": "@{triggerBody()?['text']}", "outmeetingtitle": "@{triggerBody()?['text_1']}", "outseriesmasterid": "@{triggerBody()?['text_2']}", "outpagehtml": "@{triggerBody()?['text_3']}", "outspitemcount": "@{int(coalesce(outputs('Compose_SP_Item_Count'), 0))}", "outmatchcount": "@{variables('varFinalMatchCount')}", "outbranchresult": "@{variables('varFinalMatchCount')}", "outonenoteresolverresult": "@{variables('varOneNoteResolverResult')}", "outtargetsectionpagesurl": "@{variables('varTargetSectionPagesUrl')}", "outcreatedpagelink": "@{variables('varOutputPageLink')}", "outcreatedpageselfurl": "@{variables('varOutputPageSelfUrl')}", "outfinaltargetsectionpagesurl": "@{variables('varTargetSectionPagesUrl')}", "outresolverresult": "@{variables('varOneNoteResolverResult')}", "outexistingpageselfurl": "@{variables('varFinalExistingPageSelfUrl')}", "outpagedecision": "@{variables('varFinalPageDecision')}", "outpageroute": "@{equals(variables('varFinalPageDecision'), 'PAGE_EXISTS')}", "outpageaction": "@{variables('varPageAction')}", "outupdatehtmlfragment": "@{outputs('Compose_UpdateHtmlFragment')}", "outagentresponsesummary": "@{outputs('Compose_AgentResponseSummary')}", "outstatus": "@{variables('varOutStatus')}"
    }
  },
  "runAfter": { "Set_varOutStatus": ["SUCCEEDED"] }
}
```

---

*Backup captured 16 August 2026, immediately prior to starting the permanent page-title fix (setting `title` on `Create_OneNote_Page` / `Create_Page_OneOff`). Restore from this file via Peek Code / Parameters tab if corruption strikes mid-session — do not attempt to reconstruct from memory or partial screenshots.*
