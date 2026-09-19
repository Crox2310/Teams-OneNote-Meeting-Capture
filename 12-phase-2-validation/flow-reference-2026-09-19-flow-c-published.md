# Flow C — code-view snapshot, 19 Sep 2026 (~20:32)

**Flow:** PA - Meeting Capture - C - Chat Capture
**Flow ID:** 7b295cc2-80a5-f111-b8de-7ced8d745465
**State at capture: PUBLISHED** (per handover-2026-09-19-late-urgent.md).
Source: per-action "Peek code" views pasted into chat, plus 4 designer screenshots. Action keys are taken from runAfter/output references.

Action order (designer): Trigger → FC00a → FC00b → FC01 → FC02 → FC03 → FC04 → FC05 → FC05a → FC05b → FC05f → FC05c (FC05g, FC05o, FC05p, FC05o2, FC05q, FC05s, FC05t) → FC06 → FC07 → FC08 → FC09 → FC10 → FC11 → FC11B → FC12 → FC15 → FC16 → Respond.

---

## Trigger — When Power Apps calls a flow (V2)

```json
{
  "type": "Request",
  "kind": "PowerAppV2",
  "inputs": {
    "schema": {
      "type": "object",
      "properties": {
        "text":   { "description": "Please enter your input", "title": "SeriesMasterId", "type": "string", "x-ms-content-hint": "TEXT", "x-ms-dynamically-added": true },
        "text_1": { "description": "Please enter your input", "title": "MeetingTitle", "type": "string", "x-ms-content-hint": "TEXT", "x-ms-dynamically-added": true },
        "text_2": { "description": "Please enter your input", "title": "OccurrenceDate", "type": "string", "x-ms-content-hint": "TEXT", "x-ms-dynamically-added": true }
      },
      "required": ["text", "text_1", "text_2"]
    }
  }
}
```

## FC00a_Compose_WindowStart

```json
{ "type": "Compose", "inputs": "@startOfDay(triggerBody()?['text_2'])", "runAfter": {} }
```

## FC00b_Compose_WindowEnd

```json
{ "type": "Compose", "inputs": "@addDays(outputs('FC00a_Compose_WindowStart'), 1)", "runAfter": { "FC00a_Compose_WindowStart": ["Succeeded"] } }
```

## FC01_Get_Mapping_Row

```json
{
  "type": "OpenApiConnection",
  "inputs": {
    "parameters": {
      "dataset": "https://jsainsbury.sharepoint.com/sites/coplt",
      "table": "186b3c9f-e758-4e85-83d5-685946614a0a",
      "$filter": "SeriesMasterId eq '@{triggerBody()?['text']}' and OccurrenceDate eq '@{triggerBody()?['text_2']}'",
      "$top": 1
    },
    "host": {
      "apiId": "/providers/Microsoft.PowerApps/apis/shared_sharepointonline",
      "connection": "shared_sharepointonline",
      "operationId": "GetItems"
    }
  },
  "runAfter": { "FC00b_Compose_WindowEnd": ["Succeeded"] }
}
```

## FC02_Compose_JoinUrl

```json
{ "type": "Compose", "inputs": "@first(body('FC01_Get_Mapping_Row')?['value'])?['JoinUrl']", "runAfter": { "FC01_Get_Mapping_Row": ["Succeeded"] } }
```

## FC03_Compose_PageSelfUrl

```json
{ "type": "Compose", "inputs": "@first(body('FC01_Get_Mapping_Row')?['value'])?['PageSelfUrl']", "runAfter": { "FC02_Compose_JoinUrl": ["Succeeded"] } }
```

## FC04_Compose_SectionPagesUrl

```json
{ "type": "Compose", "inputs": "@first(body('FC01_Get_Mapping_Row')?['value'])?['SectionPagesUrl']", "runAfter": { "FC03_Compose_PageSelfUrl": ["Succeeded"] } }
```

## FC05_Get_Online_Meeting

```json
{
  "type": "OpenApiConnection",
  "inputs": {
    "parameters": {
      "lookupType": "joinWebUrl",
      "lookupValue": "@outputs('FC02_Compose_JoinUrl')"
    },
    "host": {
      "apiId": "/providers/Microsoft.PowerApps/apis/shared_teams",
      "connection": "shared_teams",
      "operationId": "GetOnlineMeeting"
    }
  },
  "runAfter": { "FC04_Compose_SectionPagesUrl": ["Succeeded"] }
}
```

## FC05a_List_AI_Insights

```json
{
  "type": "OpenApiConnection",
  "inputs": {
    "parameters": { "meetingId": "@outputs('FC05_Get_Online_Meeting')?['body/id']" },
    "host": {
      "apiId": "/providers/Microsoft.PowerApps/apis/shared_teams",
      "connection": "shared_teams",
      "operationId": "ListAiInsights"
    }
  },
  "runAfter": { "FC05_Get_Online_Meeting": ["Succeeded"] }
}
```

## FC05b_Filter_to_Target_Occurrence

```json
{
  "type": "Query",
  "inputs": {
    "from": "@body('FC05a_List_AI_Insights')?['value']",
    "where": "@and(greaterOrEquals(item()?['createdDateTime'], startOfDay(triggerBody()?['text_2'])), less(item()?['createdDateTime'], addDays(startOfDay(triggerBody()?['text_2']), 1)))"
  },
  "runAfter": { "FC05a_List_AI_Insights": ["Succeeded"] }
}
```

## FC05f_Best_Insight_Id

```json
{ "type": "Compose", "inputs": "@last(body('FC05b_Filter_to_Target_Occurrence'))?['id']", "runAfter": { "FC05b_Filter_to_Target_Occurrence": ["Succeeded"] } }
```

## FC05c_Check_Insight_Available

```json
{
  "type": "If",
  "expression": {
    "and": [
      { "greater": [ "@length(body('FC05b_Filter_to_Target_Occurrence'))", 0 ] }
    ]
  },
  "actions": {
    "FC05g_Get_AI_Insight": {
      "type": "OpenApiConnection",
      "inputs": {
        "parameters": {
          "meetingId": "@outputs('FC05_Get_Online_Meeting')?['body/id']",
          "aiInsightId": "outputs('FC05f_Best_Insight_Id')"
        },
        "host": {
          "apiId": "/providers/Microsoft.PowerApps/apis/shared_teams",
          "connection": "shared_teams",
          "operationId": "GetAiInsight"
        }
      }
    },
    "FC05s_Compose_Insight_HTML": {
      "type": "Compose",
      "inputs": "@concat('<hr><h2>Meeting Capture</h2><h3>Summary</h3>', join(body('FC05p_Format_MeetingNotes_Rows'), ''), '<h3>Action Items</h3>', join(body('FC05q_Format_ActionItems_Rows'), ''))",
      "runAfter": { "FC05q_Format_ActionItems_Rows": ["Succeeded"] }
    },
    "FC05q_Format_ActionItems_Rows": {
      "type": "Select",
      "inputs": {
        "from": "@   outputs('FC05o2_Compose_ActionItems_Array')",
        "select": "@concat('<p><strong>', item()?['title'], ' (', item()?['ownerDisplayName'], '):</strong> ', item()?['text'], '</p>')"
      },
      "runAfter": { "FC05o2_Compose_ActionItems_Array": ["Succeeded"] }
    },
    "FC05p_Format_MeetingNotes_Rows": {
      "type": "Select",
      "inputs": {
        "from": "@   outputs('FC05o_Compose_MeetingNotes_Array')",
        "select": "@concat('<h4>', item()?['title'], '</h4><p>', item()?['text'], '</p>')"
      },
      "runAfter": { "FC05o_Compose_MeetingNotes_Array": ["Succeeded"] }
    },
    "FC05o_Compose_MeetingNotes_Array": {
      "type": "Compose",
      "inputs": "@body('FC05g_Get_AI_Insight')?['meetingNotes']",
      "runAfter": { "FC05g_Get_AI_Insight": ["Succeeded"] }
    },
    "FC05o2_Compose_ActionItems_Array": {
      "type": "Compose",
      "inputs": "@body('FC05g_Get_AI_Insight')?['actionItems']",
      "runAfter": { "FC05p_Format_MeetingNotes_Rows": ["Succeeded"] }
    },
    "FC05t_Update_Notes": {
      "type": "OpenApiConnection",
      "inputs": {
        "parameters": {
          "notebookKey": "Meeting Notes|$|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Master Archive Folder/Meeting Notes",
          "sectionId": "@outputs('FC04_Compose_SectionPagesUrl')",
          "pageId": "@last(split(outputs('FC03_Compose_PageSelfUrl'), '/'))",
          "updates": [
            { "target": "body", "action": "append", "position": "after", "content": "@outputs('FC05s_Compose_Insight_HTML')" }
          ]
        },
        "host": {
          "apiId": "/providers/Microsoft.PowerApps/apis/shared_onenote",
          "connection": "shared_onenote",
          "operationId": "UpdatePageContent"
        }
      },
      "runAfter": { "FC05s_Compose_Insight_HTML": ["Succeeded"] }
    }
  },
  "else": { "actions": {} },
  "runAfter": { "FC05f_Best_Insight_Id": ["Succeeded"] }
}
```

## FC06_Compose_ThreadId_NEW

```json
{
  "type": "Compose",
  "inputs": "@body('FC05_Get_Online_Meeting')?['chatInfo']?['threadId']",
  "runAfter": { "FC05c_Check_Insight_Available": ["Succeeded", "TimedOut", "Skipped", "Failed"] }
}
```

## FC07_Get_Chat_Messages_NEW

```json
{
  "type": "OpenApiConnection",
  "inputs": {
    "parameters": {
      "Uri": "@concat('https://graph.microsoft.com/v1.0/me/chats/', outputs('FC06_Compose_ThreadId_NEW'), '/messages?$top=50&$orderby=createdDateTime desc')",
      "Method": "GET",
      "CustomHeader1": "Content-Type: application/json",
      "ContentType": "application/json"
    },
    "host": {
      "apiId": "/providers/Microsoft.PowerApps/apis/shared_teams",
      "connection": "shared_teams",
      "operationId": "HttpRequest"
    }
  },
  "runAfter": { "FC06_Compose_ThreadId_NEW": ["Succeeded"] }
}
```

## FC08_Filter_Real_Messages_NEW

```json
{
  "type": "Query",
  "inputs": {
    "from": "@body('FC07_Get_Chat_Messages_NEW')?['value']",
    "where": "@equals(coalesce(item()?['messageType'], ''), 'message')"
  },
  "runAfter": { "FC07_Get_Chat_Messages_NEW": ["Succeeded"] }
}
```

## FC09_Filter_By_Window_NEW

```json
{
  "type": "Query",
  "inputs": {
    "from": "@body('FC08_Filter_Real_Messages_NEW')",
    "where": "@and(greaterOrEquals(ticks(coalesce(item()?['createdDateTime'], '1900-01-01T00:00:00Z')), ticks(outputs('FC00a_Compose_WindowStart'))), less(ticks(coalesce(item()?['createdDateTime'], '1900-01-01T00:00:00Z')), ticks(outputs('FC00b_Compose_WindowEnd'))))"
  },
  "runAfter": { "FC08_Filter_Real_Messages_NEW": ["Succeeded"] }
}
```

## FC10_Select_Message_Fields_NEW

```json
{
  "type": "Select",
  "inputs": {
    "from": "@body('FC09_Filter_By_Window_NEW')",
    "select": {
      "Time": "@coalesce(item()?['createdDateTime'], '')",
      "Speaker": "@coalesce(item()?['from']?['user']?['displayName'], item()?['from']?['application']?['displayName'], 'Unknown')",
      "Html": "@coalesce(item()?['body']?['content'], '')"
    }
  },
  "runAfter": { "FC09_Filter_By_Window_NEW": ["Succeeded"] }
}
```

## FC11_Sort_By_Time_NEW

```json
{ "type": "Compose", "inputs": "@sort(body('FC10_Select_Message_Fields_NEW'), 'Time')", "runAfter": { "FC10_Select_Message_Fields_NEW": ["Succeeded"] } }
```

## FC11B_Format_Message_Rows_NEW

```json
{
  "type": "Select",
  "inputs": {
    "from": "@outputs('FC11_Sort_By_Time_NEW')",
    "select": "@concat('<p><strong>', formatDateTime(item()?['Time'], 'HH:mm'), ' - ', item()?['Speaker'], ':</strong> ', item()?['Html'], '</p>')"
  },
  "runAfter": { "FC11_Sort_By_Time_NEW": ["Succeeded"] }
}
```

## FC12_Compose_Chat_Summary_NEW

```json
{ "type": "Compose", "inputs": "@concat('<hr><h2>Chat Transcript</h2>', join(body('FC11B_Format_Message_Rows_NEW'), ''))", "runAfter": { "FC11B_Format_Message_Rows_NEW": ["Succeeded"] } }
```

## FC15_Update_Chat

```json
{
  "type": "OpenApiConnection",
  "inputs": {
    "parameters": {
      "notebookKey": "Meeting Notes|$|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Master Archive Folder/Meeting Notes",
      "sectionId": "@outputs('FC04_Compose_SectionPagesUrl')",
      "pageId": "@last(split(outputs('FC03_Compose_PageSelfUrl'), '/'))",
      "updates": [
        { "target": "body", "action": "append", "position": "after", "content": "@outputs('FC12_Compose_Chat_Summary_NEW')" }
      ]
    },
    "host": {
      "apiId": "/providers/Microsoft.PowerApps/apis/shared_onenote",
      "connection": "shared_onenote",
      "operationId": "UpdatePageContent"
    }
  },
  "runAfter": { "FC12_Compose_Chat_Summary_NEW": ["Succeeded"] }
}
```

## FC16_Set_ChatCaptured

```json
{
  "type": "OpenApiConnection",
  "inputs": {
    "parameters": {
      "dataset": "https://jsainsbury.sharepoint.com/sites/coplt",
      "table": "186b3c9f-e758-4e85-83d5-685946614a0a",
      "id": "@first(body('FC01_Get_Mapping_Row')?['value'])?['ID']",
      "item/Status/Value": "Active",
      "item/ChatCaptured": true
    },
    "host": {
      "apiId": "/providers/Microsoft.PowerApps/apis/shared_sharepointonline",
      "connection": "shared_sharepointonline",
      "operationId": "PatchItem"
    }
  },
  "runAfter": { "FC15_Update_Chat": ["Succeeded"] }
}
```

## Respond to a Power App or flow

```json
{
  "type": "Response",
  "kind": "PowerApp",
  "inputs": {
    "schema": {
      "type": "object",
      "properties": {
        "capturepath": { "title": "CapturePath", "type": "string", "x-ms-content-hint": "TEXT", "x-ms-dynamically-added": true }
      },
      "additionalProperties": {}
    },
    "statusCode": 200,
    "body": { "capturepath": "Deferred" }
  },
  "runAfter": { "FC16_Set_ChatCaptured": ["Succeeded"] }
}
```

---

*Snapshot only — no analysis. Findings from the end-to-end review will be filed separately.*
