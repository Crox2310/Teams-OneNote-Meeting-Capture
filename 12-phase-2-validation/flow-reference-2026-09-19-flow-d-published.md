# Flow D — code-view snapshot, 19 Sep 2026 (~20:33)

**Flow:** PA - Meeting Capture - D - Auto Scheduler
**Flow ID:** d5faeba7-ee32-49dc-edb7-59f0a05ea3db
**State at capture: PUBLISHED** (designer header shows "Published").
Source: per-action "Peek code" views pasted into chat, plus 1 designer screenshot. Action keys are taken from runAfter/output references.

Action order (designer): Recurrence → FD01 → FD03 → FD04 → FD06 (FD06c → True: FD06d; False: 0 actions).

---

## Recurrence

```json
{
  "type": "Recurrence",
  "recurrence": {
    "frequency": "Minute",
    "interval": "10",
    "startTime": "2026-09-17T09:00:00.000Z"
  },
  "runtimeConfiguration": {
    "concurrency": { "runs": 1 }
  }
}
```

## FD01_—_Initialize_varOffsetMinutes

```json
{
  "type": "InitializeVariable",
  "inputs": {
    "variables": [
      { "name": "varOffsetMinutes", "type": "integer", "value": 1 }
    ]
  },
  "runAfter": {}
}
```

## FD03_—_Get_Mapping_Rows

```json
{
  "type": "OpenApiConnection",
  "inputs": {
    "parameters": {
      "dataset": "https://jsainsbury.sharepoint.com/sites/coplt",
      "table": "186b3c9f-e758-4e85-83d5-685946614a0a",
      "$filter": "OccurrenceDate eq '@{formatDateTime(convertFromUtc(utcNow(), 'GMT Standard Time'), 'yyyy-MM-dd')}' or OccurrenceDate eq '@{formatDateTime(addDays(convertFromUtc(utcNow(), 'GMT Standard Time'), -1), 'yyyy-MM-dd')}'",
      "$top": 50
    },
    "host": {
      "apiId": "/providers/Microsoft.PowerApps/apis/shared_sharepointonline",
      "connection": "shared_sharepointonline",
      "operationId": "GetItems"
    }
  },
  "runAfter": { "FD01_—_Initialize_varOffsetMinutes": ["Succeeded"] }
}
```

## FD04_—_Filter_Uncaptured_Rows

```json
{
  "type": "Query",
  "inputs": {
    "from": "@outputs('FD03_—_Get_Mapping_Rows')?['body/value']",
    "where": "@and(not(equals(item()?['ChatCaptured'], true)), not(empty(coalesce(item()?['SeriesMasterId'], ''))), not(empty(coalesce(item()?['EndTime'], ''))))"
  },
  "runAfter": { "FD03_—_Get_Mapping_Rows": ["Succeeded"] }
}
```

## FD06_—_For_Each_Uncaptured_Row

```json
{
  "type": "Foreach",
  "foreach": "@body('FD04_—_Filter_Uncaptured_Rows')",
  "actions": {
    "FD06c_—_Has_Offset_Elapsed": {
      "type": "If",
      "expression": {
        "and": [
          {
            "greaterOrEquals": [
              "@ticks(utcNow())",
              "@ticks(addMinutes(items('FD06_—_For_Each_Uncaptured_Row')?['EndTime'], variables('varOffsetMinutes')))"
            ]
          }
        ]
      },
      "actions": {
        "FD06d_—_Run_Flow_C": {
          "type": "Workflow",
          "inputs": {
            "host": { "workflowReferenceName": "7b295cc2-80a5-f111-b8de-7ced8d745465" },
            "body": {
              "text": "@items('FD06_—_For_Each_Uncaptured_Row')?['SeriesMasterId']",
              "text_1": "@items('FD06_—_For_Each_Uncaptured_Row')?['MeetingTitle']",
              "text_2": "@items('FD06_—_For_Each_Uncaptured_Row')?['OccurrenceDate']"
            }
          }
        }
      },
      "else": { "actions": {} }
    }
  },
  "runAfter": { "FD04_—_Filter_Uncaptured_Rows": ["Succeeded"] },
  "runtimeConfiguration": {
    "concurrency": { "repetitions": 1 }
  }
}
```

---

*Snapshot only — no analysis. Findings from the end-to-end review will be filed separately.*
