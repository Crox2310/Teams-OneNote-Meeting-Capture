# Flow B — code-view snapshot, 19 Sep 2026 (~20:21–20:28)

**Flow:** PA - Meeting Capture - B - Resolve OneNote Section
**State at capture: UNPUBLISHED DRAFT.** Captured mid-recovery from the 30-action wipe (triggered by deleting `Set_varTargetSectionPagesUrl_OneOffFixed` from the D2 branch). Handover recorded 29/30 values restored; `Set_varOutStatus` shown below as captured.
**Do not treat as known-good.** This is a reference snapshot only. Flow checker status at capture: not recorded.

Source: per-action "Peek code" views pasted into chat, plus 20 designer screenshots. Top-level action order below follows the designer canvas. Action keys are taken from runAfter references where the pasted view did not include the key.

---

## Trigger — When an agent calls the flow

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
        "text":   { "description": "Please enter your input", "title": "IsRecurring", "type": "string", "x-ms-content-hint": "TEXT", "x-ms-dynamically-added": true },
        "text_5": { "description": "Please enter your input", "title": "OccurrenceDate", "type": "string", "x-ms-content-hint": "TEXT", "x-ms-dynamically-added": true },
        "text_6": { "description": "Please enter your input", "title": "EndTime", "type": "string", "x-ms-content-hint": "TEXT", "x-ms-dynamically-added": true },
        "text_7": { "description": "Please enter your input", "title": "JoinUrl", "type": "string", "x-ms-content-hint": "TEXT", "x-ms-dynamically-added": true }
      },
      "required": ["text_1", "text_2", "text_3", "text_4", "text"]
    }
  },
  "metadata": {
    "flowSystemMetadata": { "flowKind": "Stateful" },
    "operationMetadataId": "23c45a79-019d-4509-b1c4-bef16d533af5"
  }
}
```

## Get_items

```json
{
  "type": "OpenApiConnection",
  "inputs": {
    "parameters": {
      "dataset": "https://jsainsbury.sharepoint.com/sites/coplt",
      "table": "186b3c9f-e758-4e85-83d5-685946614a0a",
      "$top": 500
    },
    "host": {
      "apiId": "/providers/Microsoft.PowerApps/apis/shared_sharepointonline",
      "connection": "shared_sharepointonline",
      "operationId": "GetItems"
    }
  },
  "runAfter": {},
  "metadata": { "operationMetadataId": "b592ff03-ad94-48fb-a58b-012d2bee21dd" }
}
```

## Variable initialisations (in order)

```json
{ "type": "InitializeVariable", "inputs": { "variables": [ { "name": "varFinalExistingPageSelfUrl", "type": "string" } ] }, "runAfter": { "Get_items": ["Succeeded"] }, "metadata": { "operationMetadataId": "f53f64cf-b27a-49c2-830e-184fe97ad7d3" } }
```
```json
{ "type": "InitializeVariable", "inputs": { "variables": [ { "name": "varFinalPageDecision", "type": "string" } ] }, "runAfter": { "varFinalExistingPageSelfUrl": ["Succeeded"] }, "metadata": { "operationMetadataId": "d1959a74-293b-4898-9f4b-0fca5e8ec6cf" } }
```
```json
{ "type": "InitializeVariable", "inputs": { "variables": [ { "name": "varFinalMatchCount", "type": "string" } ] }, "runAfter": { "varFinalPageDecision": ["Succeeded"] }, "metadata": { "operationMetadataId": "3bdb2daf-a77e-4031-9d87-1c05d4cfb1d7" } }
```
```json
{ "type": "InitializeVariable", "inputs": { "variables": [ { "name": "varOutStatus", "type": "string" } ] }, "runAfter": { "varFinalMatchCount": ["Succeeded"] }, "metadata": { "operationMetadataId": "e990b275-c4a2-47b5-ac36-383ee9ffd19d" } }
```
```json
{ "type": "InitializeVariable", "inputs": { "variables": [ { "name": "varOutputPageLink", "type": "string" } ] }, "runAfter": { "varOutStatus": ["Succeeded"] }, "metadata": { "operationMetadataId": "48086dc7-3fa2-48e4-b692-471b31453e2a" } }
```
```json
{ "type": "InitializeVariable", "inputs": { "variables": [ { "name": "varOutputPageSelfUrl", "type": "string" } ] }, "runAfter": { "varOutputPageLink": ["Succeeded"] }, "metadata": { "operationMetadataId": "85d9ae6e-29dd-4aea-8a21-309f92e6244c" } }
```
```json
{ "type": "InitializeVariable", "inputs": { "variables": [ { "name": "varTargetSectionPagesUrl", "type": "string" } ] }, "runAfter": { "varOutputPageSelfUrl": ["Succeeded"] }, "metadata": { "operationMetadataId": "7d342529-f620-44f1-9e53-f4196b8cc6eb" } }
```
```json
{ "type": "InitializeVariable", "inputs": { "variables": [ { "name": "varOneNoteResolverResult", "type": "string" } ] }, "runAfter": { "varTargetSectionPagesUrl": ["Succeeded"] }, "metadata": { "operationMetadataId": "1305b10b-b08e-40cf-8c80-ae9a917c780c" } }
```
```json
{ "type": "InitializeVariable", "inputs": { "variables": [ { "name": "varPageAction", "type": "string" } ] }, "runAfter": { "varOneNoteResolverResult": ["Succeeded"] }, "metadata": { "operationMetadataId": "8f508cbb-fec4-4984-b098-fb4745fcd3f3" } }
```

## UJ3b_Filter_Stale_Rows

```json
{
  "type": "Query",
  "inputs": {
    "from": "@body('Get_items')?['value']",
    "where": "@empty(item()?['SectionPagesUrl'])"
  },
  "runAfter": { "varPageAction": ["Succeeded"] }
}
```

## UJ3b_Delete_Stale_Rows

```json
{
  "type": "Foreach",
  "foreach": "@body('UJ3b_Filter_Stale_Rows')",
  "actions": {
    "Delete_item": {
      "type": "OpenApiConnection",
      "inputs": {
        "parameters": {
          "dataset": "https://jsainsbury.sharepoint.com/sites/coplt",
          "table": "186b3c9f-e758-4e85-83d5-685946614a0a",
          "id": "@items('UJ3b_Delete_Stale_Rows')?['ID']"
        },
        "host": {
          "apiId": "/providers/Microsoft.PowerApps/apis/shared_sharepointonline",
          "connection": "shared_sharepointonline",
          "operationId": "DeleteItem"
        }
      }
    }
  },
  "runAfter": { "UJ3b_Filter_Stale_Rows": ["SUCCEEDED"] }
}
```

## Condition_IsRecurring

```json
{
  "type": "If",
  "expression": {
    "and": [
      { "equals": [ "@equals(toLower(string(triggerBody()?['text'])), 'true')", true ] }
    ]
  },
  "actions": {
    "Compose_Input_SeriesMasterId": {
      "type": "Compose",
      "inputs": "@triggerBody()?['text_2']",
      "metadata": { "operationMetadataId": "96b10246-7448-4ae5-9d16-bffc855a0e77" }
    },
    "Compose_Input_MeetingTitle": {
      "type": "Compose",
      "inputs": "@triggerBody()?['text_1']",
      "runAfter": { "Compose_Input_SeriesMasterId": ["Succeeded"] },
      "metadata": { "operationMetadataId": "5e27b13e-3039-4329-83a1-1022929f608c" }
    },
    "Filter_Existing_Mapping": {
      "type": "Query",
      "inputs": {
        "from": "@body('Get_items')?['value']",
        "where": "@and(not(empty(triggerBody()?['text_2'])), equals(item()?['SeriesMasterId'],triggerBody()?['text_2']),equals(item()?['OccurrenceDate'],triggerBody()?['text_5']))"
      },
      "runAfter": { "Compose_Input_MeetingTitle": ["Succeeded"] },
      "metadata": { "operationMetadataId": "7947b646-658f-4e4f-a4e0-c09be64ff62e" }
    },
    "Compose_ExistingPageSelfUrl": {
      "type": "Compose",
      "inputs": "@if(\n  greater(length(body('Filter_Existing_Mapping')), 0),\n  first(body('Filter_Existing_Mapping'))?['PageSelfUrl'],\n  ''\n)",
      "runAfter": { "Filter_Existing_Mapping": ["Succeeded"] },
      "metadata": { "operationMetadataId": "7f4acab6-f351-443f-b499-04e59611918d" }
    },
    "Compose_PageDecision": {
      "type": "Compose",
      "inputs": "@if(\n  not(empty(outputs('Compose_ExistingPageSelfUrl'))),\n  'PAGE_EXISTS',\n  'PAGE_NOT_FOUND'\n)",
      "runAfter": { "Compose_ExistingPageSelfUrl": ["Succeeded"] },
      "metadata": { "operationMetadataId": "58b405cc-1dd3-481b-bd82-a40cef4aa8b2" }
    },
    "Compose_Match_Count": {
      "type": "Compose",
      "inputs": "@length(body('Filter_Existing_Mapping'))",
      "runAfter": { "Compose_PageDecision": ["Succeeded"] },
      "metadata": { "operationMetadataId": "e2dc65cd-e5c1-4053-8f71-7faa65239918" }
    },
    "varFinalExistingPageSelfUrl_1": {
      "type": "SetVariable",
      "inputs": { "name": "varFinalExistingPageSelfUrl", "value": "@outputs('Compose_ExistingPageSelfUrl')" },
      "runAfter": { "Compose_Match_Count": ["Succeeded"] },
      "metadata": { "operationMetadataId": "8f7109be-a108-4fad-b85a-2a283e0e647f" }
    },
    "varFinalPageDecision_1": {
      "type": "SetVariable",
      "inputs": { "name": "varFinalPageDecision", "value": "@outputs('Compose_PageDecision')" },
      "runAfter": { "varFinalExistingPageSelfUrl_1": ["Succeeded"] },
      "metadata": { "operationMetadataId": "d3d5d699-76b3-4945-818f-b475cc064cbe" }
    },
    "varFinalMatchCount_1": {
      "type": "SetVariable",
      "inputs": { "name": "varFinalMatchCount", "value": "@string(outputs('Compose_Match_Count'))" },
      "runAfter": { "varFinalPageDecision_1": ["Succeeded"] },
      "metadata": { "operationMetadataId": "49007204-f012-4063-acd9-1f68a529101a" }
    }
  },
  "else": {
    "actions": {
      "FB-F01_—_Compose_Input_MeetingTitle_(one-off)": {
        "type": "Compose",
        "inputs": "@if(empty(trim(coalesce(triggerBody()?['text_1'], ''))), 'Evt Mtg - Untitled Meeting', concat('Evt Mtg - ', substring(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(triggerBody()?['text_1'], '/', '-'), ':', '-'), '&', 'and'), '?', ''), '*', ''), '<', ''), '>', ''), '\"', ''), '|', ''), '#', ''), '''', ''), '%', ''), '~', ''), 0, min(37, length(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(triggerBody()?['text_1'], '/', '-'), ':', '-'), '&', 'and'), '?', ''), '*', ''), '<', ''), '>', ''), '\"', ''), '|', ''), '#', ''), '''', ''), '%', ''), '~', ''))))))",
        "metadata": { "operationMetadataId": "1fadaf74-38a0-4cf9-8dc4-628c98a4b563" }
      },
      "Get_Sections_OneOff": {
        "type": "OpenApiConnection",
        "inputs": {
          "parameters": {
            "notebookKey": "Meeting Notes|$|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Meeting Notes"
          },
          "host": {
            "apiId": "/providers/Microsoft.PowerApps/apis/shared_onenote",
            "connection": "shared_onenote-1",
            "operationId": "GetSectionsInNotebook"
          }
        },
        "runAfter": { "FB-F01_—_Compose_Input_MeetingTitle_(one-off)": ["Succeeded"] },
        "metadata": { "operationMetadataId": "a4489d6c-f97e-453f-a83b-8fb5f676baec" }
      },
      "Filter_OneNote_Section_OneOff": {
        "type": "Query",
        "inputs": {
          "from": "@outputs('Get_Sections_OneOff')?['body/value']",
          "where": "@equals(item()?['name'], outputs('FB-F01_—_Compose_Input_MeetingTitle_(one-off)'))"
        },
        "runAfter": { "Get_Sections_OneOff": ["Succeeded"] },
        "metadata": { "operationMetadataId": "bf078ab7-19a7-4c85-9bb1-68899ef8c925" }
      },
      "Compose_Section_Match_Count_OneOff": {
        "type": "Compose",
        "inputs": "@length(body('Filter_OneNote_Section_OneOff'))",
        "runAfter": { "Compose_SectionMatchCount_OneOff": ["Succeeded"] },
        "metadata": { "operationMetadataId": "1985366a-07f6-4317-8409-64aff8eb981a" }
      },
      "Condition_Section_Exists_OneOff": {
        "type": "If",
        "expression": {
          "and": [
            { "equals": [ "@greater(outputs('Compose_Section_Match_Count_OneOff'), 0)", "@true" ] }
          ]
        },
        "actions": {
          "For_each_1": {
            "type": "Foreach",
            "foreach": "@body('Filter_OneNote_Section_OneOff')",
            "actions": {
              "Set_varTargetSectionPagesUrl_OneOff_Exists": {
                "type": "SetVariable",
                "inputs": { "name": "varTargetSectionPagesUrl", "value": "@items('For_each_1')?['pagesUrl']" },
                "metadata": { "operationMetadataId": "12cdc792-d4e1-4ad9-8cca-c9a9df3d28a2" }
              },
              "Set_varOneNoteResolverResult_Exists_OneOff": {
                "type": "SetVariable",
                "inputs": { "name": "varOneNoteResolverResult", "value": "ExistingSection" },
                "runAfter": { "Set_varTargetSectionPagesUrl_OneOff_Exists": ["Succeeded"] },
                "metadata": { "operationMetadataId": "25d41669-c0ed-4fad-aea9-14d6232a6567" }
              }
            },
            "metadata": { "operationMetadataId": "861a9716-a0bc-479c-ac97-e5f81a2d6b19" }
          }
        },
        "else": {
          "actions": {
            "Create_Section_OneOff": {
              "type": "OpenApiConnection",
              "inputs": {
                "parameters": {
                  "body/name": "@outputs('FB-F01_—_Compose_Input_MeetingTitle_(one-off)')",
                  "notebookKey": "Meeting Notes|$|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Meeting Notes"
                },
                "host": {
                  "apiId": "/providers/Microsoft.PowerApps/apis/shared_onenote",
                  "connection": "shared_onenote-1",
                  "operationId": "CreateSectionInNotebook"
                }
              },
              "metadata": { "operationMetadataId": "d6fbc321-afea-4cf1-aaa7-aa5fedd3ba33" }
            },
            "Set_varTargetSectionPagesUrl_OneOff_Created": {
              "type": "SetVariable",
              "inputs": { "name": "varTargetSectionPagesUrl", "value": "@outputs('Create_Section_OneOff')?['body']?['pagesUrl']" },
              "runAfter": { "Create_Section_OneOff": ["Succeeded"] },
              "metadata": { "operationMetadataId": "15f484c7-4440-47d0-b0c2-60ef4aadc8f6" }
            },
            "Set_varOneNoteResolverResult_Created_OneOff": {
              "type": "SetVariable",
              "inputs": { "name": "varOneNoteResolverResult", "value": "CreatedSection" },
              "runAfter": { "Set_varTargetSectionPagesUrl_OneOff_Created": ["Succeeded"] },
              "metadata": { "operationMetadataId": "db2fd4a2-53ba-4bdb-83fa-e0529a895dfd" }
            }
          }
        },
        "runAfter": { "Compose_Section_Match_Count_OneOff": ["Succeeded"] },
        "metadata": { "operationMetadataId": "26a17b25-ab4d-47de-b59f-cae39189c1f7" }
      },
      "OF01_—_Filter_Existing_Mapping_OneOff": {
        "type": "Query",
        "inputs": {
          "from": "@body('Get_items')?['value']",
          "where": "@equals(item()?['MeetingId'],triggerBody()?['text_4'])"
        },
        "runAfter": { "Condition_Section_Exists_OneOff": ["Succeeded"] },
        "metadata": { "operationMetadataId": "77725d14-9739-40bf-a2aa-f84ab3f7b4c1" }
      },
      "OF02_—_Compose_ExistingPageSelfUrl_OneOff": {
        "type": "Compose",
        "inputs": "@if(greater(length(body('OF01_—_Filter_Existing_Mapping_OneOff')), 0), first(body('OF01_—_Filter_Existing_Mapping_OneOff'))?['PageSelfUrl'], '')",
        "runAfter": { "OF01_—_Filter_Existing_Mapping_OneOff": ["Succeeded"] },
        "metadata": { "operationMetadataId": "d3e77388-610b-47bc-8bed-00f190e86874" }
      },
      "OF03_—_Compose_PageDecision_OneOff": {
        "type": "Compose",
        "inputs": "@if(not(empty(outputs('OF02_—_Compose_ExistingPageSelfUrl_OneOff'))), 'PAGE_EXISTS', 'PAGE_NOT_FOUND')",
        "runAfter": { "OF02_—_Compose_ExistingPageSelfUrl_OneOff": ["Succeeded"] },
        "metadata": { "operationMetadataId": "5ea76696-5f61-4162-b6e0-2bf479a45c0d" }
      },
      "OF04_—_Compose_Match_Count_OneOff": {
        "type": "Compose",
        "inputs": "@length(body('OF01_—_Filter_Existing_Mapping_OneOff'))",
        "runAfter": { "OF03_—_Compose_PageDecision_OneOff": ["Succeeded"] },
        "metadata": { "operationMetadataId": "df677b1f-5c12-452b-a81b-6f78b2dc2a25" }
      },
      "OF05a_—_Set_varFinalExistingPageSelfUrl_(OneOff)": {
        "type": "SetVariable",
        "inputs": { "name": "varFinalExistingPageSelfUrl", "value": "@outputs('OF02_—_Compose_ExistingPageSelfUrl_OneOff')" },
        "runAfter": { "OF04_—_Compose_Match_Count_OneOff": ["Succeeded"] },
        "metadata": { "operationMetadataId": "f21b3096-3dd0-4d22-823a-42595226dec2" }
      },
      "OF05b_—_Set_varFinalPageDecision_(OneOff)": {
        "type": "SetVariable",
        "inputs": { "name": "varFinalPageDecision", "value": "@outputs('OF03_—_Compose_PageDecision_OneOff')" },
        "runAfter": { "OF05a_—_Set_varFinalExistingPageSelfUrl_(OneOff)": ["Succeeded"] },
        "metadata": { "operationMetadataId": "489ec848-ae43-46d1-8dae-6284cdec1c5d" }
      },
      "OF05c_—_Set_varFinalMatchCount_(OneOff)": {
        "type": "SetVariable",
        "inputs": { "name": "varFinalMatchCount", "value": "@string(outputs('OF04_—_Compose_Match_Count_OneOff'))" },
        "runAfter": { "OF05b_—_Set_varFinalPageDecision_(OneOff)": ["Succeeded"] },
        "metadata": { "operationMetadataId": "45aceba2-2324-4d67-a6b1-8da15dde0289" }
      },
      "Compose_SectionMatchCount_OneOff": {
        "type": "Compose",
        "inputs": "@string(length(body('Filter_OneNote_Section_OneOff')))",
        "runAfter": { "Filter_OneNote_Section_OneOff": ["Succeeded"] }
      }
    }
  },
  "runAfter": { "UJ3b_Delete_Stale_Rows": ["SUCCEEDED"] },
  "metadata": { "operationMetadataId": "88a540e4-abe2-456b-862d-2cd4ba59aa1a" }
}
```

## Condition_Mapping_Exists

```json
{
  "type": "If",
  "expression": {
    "and": [
      { "equals": [ "@greater(int(if(empty(variables('varFinalMatchCount')), '0', variables('varFinalMatchCount'))), 0)", "@true" ] }
    ]
  },
  "actions": {
    "Compose_Branch_Result": {
      "type": "Compose",
      "inputs": "EXISTS",
      "runAfter": { "Compose_PageRoute_Exists": ["Succeeded"] },
      "metadata": { "operationMetadataId": "35692f39-2ce6-40e9-8004-4ee5f0b87428" }
    },
    "Set_varOneNoteResolverResult_ExistingMapping": {
      "type": "SetVariable",
      "inputs": { "name": "varOneNoteResolverResult", "value": "ExistingMapping" },
      "runAfter": { "Condition_Recurring_TargetSection": ["Succeeded"] },
      "metadata": { "operationMetadataId": "cd4b5042-7849-4dbf-ade2-7118022f2031" }
    },
    "Compose_PageRoute_Exists": {
      "type": "Compose",
      "inputs": "PAGE_EXISTS_ROUTE",
      "metadata": { "operationMetadataId": "f0110054-55b9-4917-9ffb-07c5960ccd0d" }
    },
    "Condition_Recurring_TargetSection": {
      "type": "If",
      "expression": {
        "and": [
          { "equals": [ "@equals(toLower(string(triggerBody()?['text'])), 'true')", true ] }
        ]
      },
      "actions": {
        "Set_varTargetSectionPagesUrl_ExistingMapping": {
          "type": "SetVariable",
          "inputs": { "name": "varTargetSectionPagesUrl", "value": "@first(body('Filter_Existing_Mapping'))?['SectionPagesUrl']" },
          "metadata": { "operationMetadataId": "09dd411d-c75d-41d1-9b26-03621bb83196" }
        }
      },
      "else": { "actions": {} },
      "runAfter": { "Compose_Branch_Result": ["Succeeded"] },
      "metadata": { "operationMetadataId": "4a8e7a28-f4e1-418d-818e-3b661b9dfc9a" }
    }
  },
  "else": {
    "actions": {
      "Compose_Branch_Result_NoMatch": {
        "type": "Compose",
        "inputs": "CREATE_REQUIRED",
        "runAfter": { "Compose_PageRoute_CreateRequired": ["Succeeded"] },
        "metadata": { "operationMetadataId": "e3ebbaa0-c58f-413d-9338-287b1efe0b98" }
      },
      "Condition_Should_Write_Mapping": {
        "type": "If",
        "expression": {
          "and": [
            { "equals": [ "@equals(toLower(string(triggerBody()?['text'])), 'true')", "@true" ] }
          ]
        },
        "actions": {
          "Compose_MappingWriteSucceeded": {
            "type": "Compose",
            "inputs": "@if(equals(outputs('Create_Mapping_Item_Recurring')?['statusCode'], 201), 'true', 'false')",
            "runAfter": { "Create_Mapping_Item_Recurring": ["Succeeded", "TimedOut", "Skipped", "Failed"] }
          },
          "Create_Mapping_Item_Recurring": {
            "type": "OpenApiConnection",
            "inputs": {
              "parameters": {
                "dataset": "https://jsainsbury.sharepoint.com/sites/coplt",
                "table": "186b3c9f-e758-4e85-83d5-685946614a0a",
                "item/Title": "Mapping",
                "item/SeriesMasterId": "@outputs('Compose_Input_SeriesMasterId')",
                "item/MeetingTitle": "@outputs('Compose_Input_MeetingTitle')",
                "item/SectionPagesUrl": "@variables('varTargetSectionPagesUrl')",
                "item/Status/Value": "Active",
                "item/OccurrenceDate": "@triggerBody()?['text_5']",
                "item/JoinUrl": "@trim(coalesce(triggerBody()?['text_7'], ''))",
                "item/EndTime": "@replace(coalesce(triggerBody()?['text_6'], ''), 'UTC|', '')"
              },
              "host": {
                "apiId": "/providers/Microsoft.PowerApps/apis/shared_sharepointonline",
                "connection": "shared_sharepointonline",
                "operationId": "PostItem"
              }
            }
          }
        },
        "else": { "actions": {} },
        "runAfter": { "Condition_Section_Exists_Recurring": ["Succeeded"] },
        "metadata": { "operationMetadataId": "8293f5f1-aaaa-4bb9-a154-4c489fcfd781" }
      },
      "Compose_PageRoute_CreateRequired": {
        "type": "Compose",
        "inputs": "PAGE_NOT_FOUND_ROUTE",
        "runAfter": { "Compose_IgnoreSeriesMasterId": ["Succeeded"] },
        "metadata": { "operationMetadataId": "e3eec170-64b0-4095-aea8-7c3eac8581fe" }
      },
      "Compose_IgnoreSeriesMasterId": {
        "type": "Compose",
        "inputs": "''",
        "metadata": { "operationMetadataId": "83d743e1-3eed-4101-b407-cabdef289d58" }
      },
      "Compose_SectionDisplayName": {
        "type": "Compose",
        "inputs": "@triggerBody()?['text_1']",
        "runAfter": { "Compose_Branch_Result_NoMatch": ["Succeeded"] },
        "metadata": { "operationMetadataId": "4c40d305-dd77-4973-a2eb-6558fc932bc2" }
      },
      "Compose_SafeSectionName": {
        "type": "Compose",
        "inputs": "@if(empty(trim(coalesce(outputs('Compose_SectionDisplayName'), ''))), 'Mtg - Untitled Meeting', concat('Mtg - ', substring(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(outputs('Compose_SectionDisplayName'), '/', '-'), ':', '-'), '&', 'and'), '?', ''), '*', ''), '<', ''), '>', ''), '\"', ''), '|', ''), '#', ''), '''', ''), '%', ''), '~', ''), 0, min(43, length(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(outputs('Compose_SectionDisplayName'), '/', '-'), ':', '-'), '&', 'and'), '?', ''), '*', ''), '<', ''), '>', ''), '\"', ''), '|', ''), '#', ''), '''', ''), '%', ''), '~', ''))))))",
        "runAfter": { "Compose_SectionDisplayName": ["Succeeded"] },
        "metadata": { "operationMetadataId": "625c3182-1b5f-4c59-9609-2ea46876c280" }
      },
      "Get_Sections_Recurring": {
        "type": "OpenApiConnection",
        "inputs": {
          "parameters": {
            "notebookKey": "Meeting Notes|$|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Meeting Notes"
          },
          "host": {
            "apiId": "/providers/Microsoft.PowerApps/apis/shared_onenote",
            "connection": "shared_onenote-1",
            "operationId": "GetSectionsInNotebook"
          }
        },
        "runAfter": { "Compose_SafeSectionName": ["Succeeded"] },
        "metadata": { "operationMetadataId": "f320a286-852a-4213-8fb9-c1162b89eda3" }
      },
      "Filter_OneNote_Section_Recurring": {
        "type": "Query",
        "inputs": {
          "from": "@outputs('Get_Sections_Recurring')?['body/value']",
          "where": "@equals(item()?['name'],outputs('Compose_SafeSectionName'))"
        },
        "runAfter": { "Get_Sections_Recurring": ["Succeeded"] },
        "metadata": { "operationMetadataId": "5a7f07af-3280-4192-a73c-51a9395d6323" }
      },
      "Condition_Section_Exists_Recurring": {
        "type": "If",
        "expression": {
          "and": [
            { "equals": [ "@equals(outputs('Compose_Section_Match_Count_Recurring'), 1)", "@true" ] }
          ]
        },
        "actions": {
          "Apply_to_each": {
            "type": "Foreach",
            "foreach": "@body('Filter_OneNote_Section_Recurring')",
            "actions": {
              "varTargetSectionPagesUrl_1": {
                "type": "SetVariable",
                "inputs": { "name": "varTargetSectionPagesUrl", "value": "@items('Apply_to_each')?['pagesUrl']" },
                "metadata": { "operationMetadataId": "60cda415-29c2-4280-8d98-048dfdcc1e68" }
              },
              "varOneNoteResolverResult_1": {
                "type": "SetVariable",
                "inputs": { "name": "varOneNoteResolverResult", "value": "ExistingSection" },
                "runAfter": { "varTargetSectionPagesUrl_1": ["Succeeded"] },
                "metadata": { "operationMetadataId": "6f4a69ab-2dc0-40d2-b378-61e02cb05ca2" }
              }
            },
            "metadata": { "operationMetadataId": "a2b5f418-47ae-43a7-8794-10b63de5f99d" }
          }
        },
        "else": {
          "actions": {
            "Condition_Section_Count_Is_Zero": {
              "type": "If",
              "expression": {
                "and": [
                  { "equals": [ "@outputs('Compose_Section_Match_Count_Recurring')", 0 ] }
                ]
              },
              "actions": {
                "Create_Section_Recurring": {
                  "type": "OpenApiConnection",
                  "inputs": {
                    "parameters": {
                      "body/name": "@outputs('Compose_SafeSectionName')",
                      "notebookKey": "Meeting Notes|$|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Meeting Notes"
                    },
                    "host": {
                      "apiId": "/providers/Microsoft.PowerApps/apis/shared_onenote",
                      "connection": "shared_onenote-1",
                      "operationId": "CreateSectionInNotebook"
                    }
                  },
                  "metadata": { "operationMetadataId": "64608b0b-3067-4f0d-8e08-a1306bbdfa83" }
                },
                "varTargetSectionPagesUrl_2": {
                  "type": "SetVariable",
                  "inputs": { "name": "varTargetSectionPagesUrl", "value": "@outputs('Create_Section_Recurring')?['body']?['pagesUrl']" },
                  "runAfter": { "Create_Section_Recurring": ["Succeeded"] },
                  "metadata": { "operationMetadataId": "8ef87e4d-2eb7-4215-8ad1-0a82607678bb" }
                },
                "varOneNoteResolverResult_2": {
                  "type": "SetVariable",
                  "inputs": { "name": "varOneNoteResolverResult", "value": "CreatedSection" },
                  "runAfter": { "varTargetSectionPagesUrl_2": ["Succeeded"] },
                  "metadata": { "operationMetadataId": "cd6e73d3-589e-43fc-9125-c93666657351" }
                }
              },
              "else": { "actions": {} }
            }
          }
        },
        "runAfter": { "Compose_Section_Match_Count_Recurring": ["Succeeded"] },
        "metadata": { "operationMetadataId": "1bbabc6b-f438-4356-9d40-7976814bebf3" }
      },
      "Compose_Section_Match_Count_Recurring": {
        "type": "Compose",
        "inputs": "@length(body('Filter_OneNote_Section_Recurring'))",
        "runAfter": { "Compose_SectionMatchCount_Recurring": ["Succeeded"] },
        "metadata": { "operationMetadataId": "1022587d-2178-4e8b-9be8-06e1398cc0b6" }
      },
      "Compose_SectionMatchCount_Recurring": {
        "type": "Compose",
        "inputs": "@string(length(body('Filter_OneNote_Section_Recurring')))",
        "runAfter": { "Filter_OneNote_Section_Recurring": ["Succeeded"] }
      }
    }
  },
  "runAfter": { "Condition_IsRecurring": ["Succeeded"] },
  "metadata": { "operationMetadataId": "256a25d3-9f83-4980-8484-a4b895a86096" }
}
```

## Condition_Should_Create_Page

```json
{
  "type": "If",
  "expression": {
    "and": [
      { "equals": [ "@equals(variables('varFinalPageDecision'), 'PAGE_NOT_FOUND')", true ] }
    ]
  },
  "actions": {
    "Create_OneNote_Page": {
      "type": "OpenApiConnection",
      "inputs": {
        "parameters": {
          "notebookKey": "Meeting Notes|$|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Meeting Notes",
          "sectionId": "@variables('varTargetSectionPagesUrl')",
          "pageContent": "<p class=\"editor-paragraph\">@{triggerBody()?['text_3']}</p>"
        },
        "host": {
          "apiId": "/providers/Microsoft.PowerApps/apis/shared_onenote",
          "connection": "shared_onenote",
          "operationId": "CreatePageInSection"
        }
      },
      "runAfter": { "Compose_SafePageTitle": ["Succeeded"] },
      "metadata": { "operationMetadataId": "e65eb2ff-57a9-4520-88b6-b938c39c88a7" }
    },
    "Compose_PageSelfUrl_Created": {
      "type": "Compose",
      "inputs": "@body('Create_OneNote_Page')?['self']",
      "runAfter": { "Delay_Post_Page_Creation": ["Succeeded"] },
      "metadata": { "operationMetadataId": "e6f2cd3b-ba97-4055-b988-0b828956a049" }
    },
    "OF09-Gate_—_Condition_Is_Recurring_(SP_Write)": {
      "type": "If",
      "expression": {
        "and": [
          { "equals": [ "@equals(toLower(string(triggerBody()?['text'])), 'true')", true ] }
        ]
      },
      "actions": {
        "HTTP_Update_SP_PageSelfUrl": {
          "type": "OpenApiConnection",
          "inputs": {
            "parameters": {
              "dataset": "https://jsainsbury.sharepoint.com/sites/coplt",
              "parameters/method": "POST",
              "parameters/uri": "_api/web/lists/GetByTitle('RecurringMeetingSectionMap')/items(@{if(greater(length(body('Filter_Existing_Mapping')),0), first(body('Filter_Existing_Mapping'))?['ID'], body('Create_Mapping_Item_Recurring')?['ID'])})",
              "parameters/headers": {
                "Accept": "application/json;odata=nometadata",
                "Content-Type": "application/json;odata=nometadata",
                "IF-MATCH": "*",
                "X-HTTP-Method": "MERGE"
              },
              "parameters/body": "{\n     \"PageSelfUrl\": \"@{outputs('Compose_PageSelfUrl_Created')}\",\n     \"PageWebUrl\": \"@{outputs('Create_OneNote_Page')?['body']?['links']?['oneNoteWebUrl']?['href']}\"\n   }"
            },
            "host": {
              "apiId": "/providers/Microsoft.PowerApps/apis/shared_sharepointonline",
              "connection": "shared_sharepointonline",
              "operationId": "HttpRequest"
            }
          },
          "metadata": { "operationMetadataId": "667fe995-0738-4a7c-b3fd-cc920da51306" }
        },
        "Set_varPageAction_Created": {
          "type": "SetVariable",
          "inputs": { "name": "varPageAction", "value": "Created" },
          "runAfter": { "HTTP_Update_SP_PageSelfUrl": ["Succeeded"] },
          "metadata": { "operationMetadataId": "3cf6f3c9-98bb-4074-9761-cf48709c7b32" }
        },
        "Set_varOutputPageSelfUrl_Created": {
          "type": "SetVariable",
          "inputs": { "name": "varOutputPageSelfUrl", "value": "@outputs('Compose_PageSelfUrl_Created')" },
          "runAfter": { "Set_varPageAction_Created": ["Succeeded"] },
          "metadata": { "operationMetadataId": "a6f1f66b-2c52-4a19-aa2d-a2c3a95a97b8" }
        },
        "Set_varOutputPageLink_Created": {
          "type": "SetVariable",
          "inputs": { "name": "varOutputPageLink", "value": "@outputs('Create_OneNote_Page')?['body']?['links']?['oneNoteWebUrl']?['href']" },
          "runAfter": { "Set_varOutputPageSelfUrl_Created": ["Succeeded"] },
          "metadata": { "operationMetadataId": "329f5994-3db5-4a88-b6c2-95b4dd3b8941" }
        }
      },
      "else": {
        "actions": {
          "OF09b-i_—_Condition_Should_Insert_Mapping_(OneOff)": {
            "type": "If",
            "expression": {
              "and": [
                { "equals": [ "@equals(length(body('OF01_—_Filter_Existing_Mapping_OneOff')), 0)", true ] }
              ]
            },
            "actions": {
              "Compose_MappingWriteSucceeded_OneOff": {
                "type": "Compose",
                "inputs": "@if(equals(outputs('Create_Mapping_Item_OneOff')?['statusCode'], 201), 'true', 'false')",
                "runAfter": { "Create_Mapping_Item_OneOff": ["Succeeded", "TimedOut", "Skipped", "Failed"] }
              },
              "Create_Mapping_Item_OneOff": {
                "type": "OpenApiConnection",
                "inputs": {
                  "parameters": {
                    "dataset": "https://jsainsbury.sharepoint.com/sites/coplt",
                    "table": "186b3c9f-e758-4e85-83d5-685946614a0a",
                    "item/Title": "Mapping",
                    "item/MeetingTitle": "@outputs('FB-F01_—_Compose_Input_MeetingTitle_(one-off)')",
                    "item/SectionPagesUrl": "@variables('varTargetSectionPagesUrl')",
                    "item/Status/Value": "Active",
                    "item/MeetingId": "@triggerBody()?['text_4']"
                  },
                  "host": {
                    "apiId": "/providers/Microsoft.PowerApps/apis/shared_sharepointonline",
                    "connection": "shared_sharepointonline",
                    "operationId": "PostItem"
                  }
                }
              }
            },
            "else": { "actions": {} },
            "metadata": { "operationMetadataId": "fb93a7c0-f0ea-44ab-a650-ad0d27ec5bd9" }
          },
          "OF09b_—_HTTP_Update_SP_PageSelfUrl_(OneOff)": {
            "type": "OpenApiConnection",
            "inputs": {
              "parameters": {
                "dataset": "https://jsainsbury.sharepoint.com/sites/coplt",
                "parameters/method": "POST",
                "parameters/uri": "_api/web/lists/GetByTitle('RecurringMeetingSectionMap')/items(@{if(greater(length(body('OF01_—_Filter_Existing_Mapping_OneOff')),0), first(body('OF01_—_Filter_Existing_Mapping_OneOff'))?['ID'], body('Create_Mapping_Item_OneOff')?['ID'])})",
                "parameters/headers": {
                  "Accept": "application/json;odata=nometadata",
                  "Content-Type": "application/json;odata=nometadata",
                  "IF-MATCH": "*",
                  "X-HTTP-Method": "MERGE"
                },
                "parameters/body": "{\n     \"PageSelfUrl\": \"@{outputs('Compose_PageSelfUrl_Created')}\",\n     \"PageWebUrl\": \"@{body('Create_OneNote_Page')?['links']?['oneNoteWebUrl']?['href']}\"\n   }"
              },
              "host": {
                "apiId": "/providers/Microsoft.PowerApps/apis/shared_sharepointonline",
                "connection": "shared_sharepointonline",
                "operationId": "HttpRequest"
              }
            },
            "runAfter": { "OF09b-i_—_Condition_Should_Insert_Mapping_(OneOff)": ["Succeeded"] },
            "metadata": { "operationMetadataId": "e5e26e5b-0e6f-48c5-b90e-5f527bb6348a" }
          },
          "Set_varPageAction_Created_OneOff": {
            "type": "SetVariable",
            "inputs": { "name": "varPageAction", "value": "Created" },
            "runAfter": { "OF09b_—_HTTP_Update_SP_PageSelfUrl_(OneOff)": ["Succeeded"] },
            "metadata": { "operationMetadataId": "f2af053a-c4ed-4cca-98ac-cf39351bdd47" }
          },
          "Set_varOutputPageSelfUrl_Created_OneOff": {
            "type": "SetVariable",
            "inputs": { "name": "varOutputPageSelfUrl", "value": "@outputs('Compose_PageSelfUrl_Created')" },
            "runAfter": { "Set_varPageAction_Created_OneOff": ["Succeeded"] },
            "metadata": { "operationMetadataId": "7be996dd-a3a5-494d-943a-6f69ae0780ae" }
          },
          "Set_varOutputPageLink_Created_OneOff_Gate": {
            "type": "SetVariable",
            "inputs": { "name": "varOutputPageLink", "value": "@outputs('Create_OneNote_Page')?['body']?['links']?['oneNoteWebUrl']?['href']" },
            "runAfter": { "Set_varOutputPageSelfUrl_Created_OneOff": ["Succeeded"] },
            "metadata": { "operationMetadataId": "eec5be87-5551-4152-8d64-663163b505a1" }
          }
        }
      },
      "runAfter": { "Set_PageTitle_Recurring": ["Succeeded"] },
      "metadata": { "operationMetadataId": "881596ea-b400-4059-b6a1-e3e82a97422b" }
    },
    "Compose_SafePageTitle": {
      "type": "Compose",
      "inputs": "@if(empty(trim(coalesce(triggerBody()?['text_1'], ''))), 'Untitled Meeting', concat(substring(replace(replace(replace(replace(triggerBody()?['text_1'], '&', 'and'), '<', ''), '>', ''), '\"', ''), 0, min(150, length(replace(replace(replace(replace(triggerBody()?['text_1'], '&', 'and'), '<', ''), '>', ''), '\"', '')))), if(empty(coalesce(triggerBody()?['text_5'], '')), '', concat(' - ', formatDateTime(triggerBody()?['text_5'], 'd MMM yyyy')))))",
      "metadata": { "operationMetadataId": "80ad50ea-df45-42a5-a266-8e6169f4e374" }
    },
    "Set_PageTitle_Recurring": {
      "type": "OpenApiConnection",
      "inputs": {
        "parameters": {
          "notebookKey": "Meeting Notes|$|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Meeting Notes",
          "sectionId": "@variables('varTargetSectionPagesUrl')",
          "pageId": "@outputs('Compose_ConfirmedCreatedPageId')",
          "updates": [
            { "target": "title", "action": "replace", "content": "@outputs('Compose_SafePageTitle')" }
          ]
        },
        "host": {
          "apiId": "/providers/Microsoft.PowerApps/apis/shared_onenote",
          "connection": "shared_onenote-1",
          "operationId": "UpdatePageContent"
        }
      },
      "runAfter": { "Compose_ConfirmedCreatedPageId": ["Succeeded"] },
      "metadata": { "operationMetadataId": "8eae5cd4-ddc2-4c7f-b834-c9ab8a58c977" }
    },
    "Get_Pages_In_Section_Recurring_PostCreate": {
      "type": "OpenApiConnection",
      "inputs": {
        "parameters": {
          "notebookKey": "Meeting Notes|$|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Meeting Notes",
          "sectionId": "@variables('varTargetSectionPagesUrl')"
        },
        "host": {
          "apiId": "/providers/Microsoft.PowerApps/apis/shared_onenote",
          "connection": "shared_onenote-1",
          "operationId": "GetPagesInSection"
        }
      },
      "runAfter": { "Compose_PageSelfUrl_Created": ["Succeeded"] },
      "metadata": { "operationMetadataId": "b912d89c-912f-4330-8e6a-2127501291f1" }
    },
    "Filter_Pages_By_SelfUrl_Recurring": {
      "type": "Query",
      "inputs": {
        "from": "@outputs('Get_Pages_In_Section_Recurring_PostCreate')?['body']?['value']",
        "where": "@equals(item()?['self'],outputs('Compose_PageSelfUrl_Created'))"
      },
      "runAfter": { "Get_Pages_In_Section_Recurring_PostCreate": ["Succeeded"] },
      "metadata": { "operationMetadataId": "5405ed9d-8e37-41b8-8908-4d95de6287d5" }
    },
    "Compose_ConfirmedCreatedPageId": {
      "type": "Compose",
      "inputs": "@if(greater(length(body('Filter_Pages_By_SelfUrl_Recurring')), 0), first(body('Filter_Pages_By_SelfUrl_Recurring'))?['id'], '')",
      "runAfter": { "Filter_Pages_By_SelfUrl_Recurring": ["Succeeded"] },
      "metadata": { "operationMetadataId": "783abe93-437f-4a0c-87f1-ee1f65a0358f" }
    },
    "Delay_Post_Page_Creation": {
      "type": "Wait",
      "inputs": { "interval": { "count": 5, "unit": "Second" } },
      "runAfter": { "Create_OneNote_Page": ["Succeeded"] },
      "metadata": { "operationMetadataId": "1dde603b-7811-45b6-a930-7181353707d2" }
    }
  },
  "else": {
    "actions": {
      "Set_varPageAction_ExistsNoCreate": {
        "type": "SetVariable",
        "inputs": { "name": "varPageAction", "value": "Updated" },
        "metadata": { "operationMetadataId": "3f94c963-2c50-475d-bbdf-85cc9de75aa5" }
      },
      "Set_varOutputPageSelfUrl_Existing": {
        "type": "SetVariable",
        "inputs": { "name": "varOutputPageSelfUrl", "value": "@variables('varFinalExistingPageSelfUrl')" },
        "runAfter": { "Set_varPageAction_ExistsNoCreate": ["Succeeded"] },
        "metadata": { "operationMetadataId": "6759df14-12e8-477f-a542-7f3f3c1e595d" }
      },
      "Compose_UpdateHtmlFragment": {
        "type": "Compose",
        "inputs": "@concat('<hr><h2>Automated update</h2><p><strong>Updated by:</strong> Meeting Capture Agent</p><p><strong>Update note:</strong> Meeting details were refreshed by the automation. Existing human-entered notes were preserved below.</p>', triggerBody()?['text_3'])",
        "runAfter": { "Set_varOutputPageSelfUrl_Existing": ["Succeeded"] },
        "metadata": { "operationMetadataId": "a616bfcb-e28f-43fd-99e2-6d2d1a22c0c0" }
      },
      "Compose_ExistingPageId": {
        "type": "Compose",
        "inputs": "@last(split(variables('varOutputPageSelfUrl'), '/'))",
        "runAfter": { "Compose_UpdateHtmlFragment": ["Succeeded"] },
        "metadata": { "operationMetadataId": "3215643d-9674-48d9-83db-4a4fbe060772" }
      },
      "Condition_Is_Genuine_Existing_Page": {
        "type": "If",
        "expression": {
          "and": [
            { "equals": [ "@contains(createArray('ExistingMapping', 'ExistingSection'), variables('varOneNoteResolverResult'))", true ] }
          ]
        },
        "actions": {
          "Get_Sections_Existing_Branch": {
            "type": "OpenApiConnection",
            "inputs": {
              "parameters": {
                "notebookKey": "Meeting Notes|$|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Meeting Notes"
              },
              "host": {
                "apiId": "/providers/Microsoft.PowerApps/apis/shared_onenote",
                "connection": "shared_onenote",
                "operationId": "GetSectionsInNotebook"
              }
            },
            "metadata": { "operationMetadataId": "9f34520a-1871-4a49-a436-bc5fba160b68" }
          },
          "Filter_Existing_Section_By_Name": {
            "type": "Query",
            "inputs": {
              "from": "@outputs('Get_Sections_Existing_Branch')?['body/value']",
              "where": "@equals(item()?['name'],outputs('Compose_SafeSectionName_ExistingBranch'))"
            },
            "runAfter": { "Compose_SafeSectionName_ExistingBranch": ["Succeeded"] },
            "metadata": { "operationMetadataId": "a8785fcf-d8ef-41cd-aec6-0b2338d12db5" }
          },
          "Compose_SectionDisplayName_ExistingBranch": {
            "type": "Compose",
            "inputs": "@triggerBody()?['text_1']",
            "runAfter": { "Get_Sections_Existing_Branch": ["Succeeded"] },
            "metadata": { "operationMetadataId": "4c26e79d-ac23-4fd7-a52d-cec89d2962d5" }
          },
          "Compose_SafeSectionName_ExistingBranch": {
            "type": "Compose",
            "inputs": "@if(empty(trim(coalesce(outputs('Compose_SectionDisplayName_ExistingBranch'), ''))), 'Mtg - Untitled Meeting', concat('Mtg - ', substring(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(outputs('Compose_SectionDisplayName_ExistingBranch'), '/', '-'), ':', '-'), '&', 'and'), '?', ''), '*', ''), '<', ''), '>', ''), '\"', ''), '|', ''), '#', ''), '''', ''), '%', ''), '~', ''), 0, min(43, length(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(outputs('Compose_SectionDisplayName_ExistingBranch'), '/', '-'), ':', '-'), '&', 'and'), '?', ''), '*', ''), '<', ''), '>', ''), '\"', ''), '|', ''), '#', ''), '''', ''), '%', ''), '~', ''))))))",
            "runAfter": { "Compose_SectionDisplayName_ExistingBranch": ["Succeeded"] },
            "metadata": { "operationMetadataId": "2a744adf-50eb-4ff1-9e73-1800f259048c" }
          },
          "Apply_to_each_Existing_Section": {
            "type": "Foreach",
            "foreach": "@body('Filter_Existing_Section_By_Name')",
            "actions": {
              "Get_Pages_In_Section_Existing_Branch": {
                "type": "OpenApiConnection",
                "inputs": {
                  "parameters": {
                    "notebookKey": "Meeting Notes|$|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Meeting Notes",
                    "sectionId": "@items('Apply_to_each_Existing_Section')?['pagesUrl']"
                  },
                  "host": {
                    "apiId": "/providers/Microsoft.PowerApps/apis/shared_onenote",
                    "connection": "shared_onenote-1",
                    "operationId": "GetPagesInSection"
                  }
                },
                "metadata": { "operationMetadataId": "c2d2915d-25af-4dc8-8e22-eed37729d310" }
              },
              "Compose_MeetingTitleForPageMatch": {
                "type": "Compose",
                "inputs": "@triggerBody()?['text_1']",
                "runAfter": { "Get_Pages_In_Section_Existing_Branch": ["Succeeded"] },
                "metadata": { "operationMetadataId": "b4b5fa97-97f3-43e4-a548-7dff6ced43fd" }
              },
              "Filter_Pages_By_Title": {
                "type": "Query",
                "inputs": {
                  "from": "@outputs('Get_Pages_In_Section_Existing_Branch')?['body']?['value']",
                  "where": "@contains(item()?['title'], formatDateTime(triggerBody()?['text_5'], 'd MMM yyyy'))"
                },
                "runAfter": { "Compose_MeetingTitleForPageMatch": ["Succeeded"] },
                "metadata": { "operationMetadataId": "c89fd423-892c-4aea-807b-e8ef70e5773e" }
              },
              "Compose_RealExistingPageId": {
                "type": "Compose",
                "inputs": "@if(greater(length(body('Filter_Pages_By_Title')), 0), first(body('Filter_Pages_By_Title'))?['id'], '')",
                "runAfter": { "Filter_Pages_By_Title": ["Succeeded"] },
                "metadata": { "operationMetadataId": "0aae6759-e808-46f5-a5c7-9d44778108b7" }
              },
              "Guard_RealPageId_Not_Empty": {
                "type": "If",
                "expression": {
                  "and": [
                    { "equals": [ "@empty(outputs('Compose_RealExistingPageId'))", true ] }
                  ]
                },
                "actions": {
                  "Guard_Create_Page_Fallback": {
                    "type": "OpenApiConnection",
                    "inputs": {
                      "parameters": {
                        "notebookKey": "Meeting Notes|$|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Meeting Notes",
                        "sectionId": "@items('Apply_to_each_Existing_Section')?['pagesUrl']",
                        "pageContent": "<p class=\"editor-paragraph\">@{triggerBody()?['text_3']}</p>"
                      },
                      "host": {
                        "apiId": "/providers/Microsoft.PowerApps/apis/shared_onenote",
                        "connection": "shared_onenote-1",
                        "operationId": "CreatePageInSection"
                      }
                    }
                  }
                },
                "else": {
                  "actions": {
                    "Guard_Update_Page_Normal": {
                      "type": "OpenApiConnection",
                      "inputs": {
                        "parameters": {
                          "notebookKey": "Meeting Notes|$|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Master Archive Folder/Meeting Notes",
                          "sectionId": "@items('Apply_to_each_Existing_Section')?['pagesUrl']",
                          "pageId": "@outputs('Compose_RealExistingPageId')",
                          "updates": [
                            { "target": "body", "action": "append", "position": "after", "content": "@outputs('Compose_UpdateHtmlFragment')" }
                          ]
                        },
                        "host": {
                          "apiId": "/providers/Microsoft.PowerApps/apis/shared_onenote",
                          "connection": "shared_onenote-1",
                          "operationId": "UpdatePageContent"
                        }
                      }
                    }
                  }
                },
                "runAfter": { "Compose_RealExistingPageId": ["Succeeded", "SKIPPED"] }
              }
            },
            "runAfter": { "Filter_Existing_Section_By_Name": ["Succeeded"] },
            "metadata": { "operationMetadataId": "80316e0e-febc-4696-b786-d7e4ed631317" }
          },
          "Set_varPageAction_UpdatedAppend": {
            "type": "SetVariable",
            "inputs": { "name": "varPageAction", "value": "Updated" },
            "runAfter": { "Apply_to_each_Existing_Section": ["Succeeded"] },
            "metadata": { "operationMetadataId": "72549575-6926-4743-9d96-b43e45e8c56d" }
          },
          "Set_varOutputPageLink_Existing": {
            "type": "SetVariable",
            "inputs": { "name": "varOutputPageLink", "value": "@first(coalesce(body('Filter_Existing_Mapping'), body('OF01_—_Filter_Existing_Mapping_OneOff'), createArray()))?['PageWebUrl']" },
            "runAfter": { "Set_varPageAction_UpdatedAppend": ["Succeeded"] },
            "metadata": { "operationMetadataId": "e3376096-56a8-4742-ac05-b0ce1eb3daca" }
          }
        },
        "else": {
          "actions": {
            "Create_Page_OneOff": {
              "type": "OpenApiConnection",
              "inputs": {
                "parameters": {
                  "notebookKey": "Meeting Notes|$|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Meeting Notes",
                  "sectionId": "@variables('varTargetSectionPagesUrl')",
                  "pageContent": "@triggerBody()?['text_3']"
                },
                "host": {
                  "apiId": "/providers/Microsoft.PowerApps/apis/shared_onenote",
                  "connection": "shared_onenote-1",
                  "operationId": "CreatePageInSection"
                }
              },
              "runAfter": { "Compose_SafeSectionName_D2": ["Succeeded"] },
              "metadata": { "operationMetadataId": "6a0a34b9-6c06-4399-9086-583cfaf6485d" }
            },
            "Set_varOutputPageLink_Created_OneOff": {
              "type": "SetVariable",
              "inputs": { "name": "varOutputPageLink", "value": "@outputs('Create_Page_OneOff')?['body']?['links']?['oneNoteWebUrl']?['href']" },
              "runAfter": { "Set_PageTitle_OneOff": ["Succeeded"] },
              "metadata": { "operationMetadataId": "d7535f25-150f-4979-ba2b-a8a4862e9243" }
            },
            "Compose_SafePageTitle_OneOff": {
              "type": "Compose",
              "inputs": "@if(empty(trim(coalesce(triggerBody()?['text_1'], ''))), 'Untitled Meeting', concat(substring(replace(replace(replace(replace(triggerBody()?['text_1'], '&', 'and'), '<', ''), '>', ''), '\"', ''), 0, min(150, length(replace(replace(replace(replace(triggerBody()?['text_1'], '&', 'and'), '<', ''), '>', ''), '\"', '')))), if(empty(coalesce(triggerBody()?['text_5'], '')), '', concat(' - ', formatDateTime(triggerBody()?['text_5'], 'd MMM yyyy')))))",
              "runAfter": { "Condition_Section_Exists_D2": ["Succeeded"] },
              "metadata": { "operationMetadataId": "679c7abe-1980-4946-9f4b-8efc106f9ae3" }
            },
            "Get_Pages_In_Section_OneOff_PostCreate": {
              "type": "OpenApiConnection",
              "inputs": {
                "parameters": {
                  "notebookKey": "Meeting Notes|$|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Meeting Notes",
                  "sectionId": "@variables('varTargetSectionPagesUrl')"
                },
                "host": {
                  "apiId": "/providers/Microsoft.PowerApps/apis/shared_onenote",
                  "connection": "shared_onenote-1",
                  "operationId": "GetPagesInSection"
                }
              },
              "runAfter": { "Delay_Post_Page_Creation_OneOff": ["Succeeded"] },
              "metadata": { "operationMetadataId": "ad229c1e-8cc9-40ea-90bd-a63659cd5296" }
            },
            "Filter_Pages_By_SelfUrl_OneOff": {
              "type": "Query",
              "inputs": {
                "from": "@outputs('Get_Pages_In_Section_OneOff_PostCreate')?['body']?['value']",
                "where": "@equals(item()?['self'],body('Create_Page_OneOff')?['self'])"
              },
              "runAfter": { "Get_Pages_In_Section_OneOff_PostCreate": ["Succeeded"] },
              "metadata": { "operationMetadataId": "61dfaba5-d009-42e7-916b-00a06c7114e6" }
            },
            "Compose_ConfirmedCreatedPageId_OneOff": {
              "type": "Compose",
              "inputs": "@if(greater(length(body('Filter_Pages_By_SelfUrl_OneOff')), 0), first(body('Filter_Pages_By_SelfUrl_OneOff'))?['id'], '')",
              "runAfter": { "Filter_Pages_By_SelfUrl_OneOff": ["Succeeded"] },
              "metadata": { "operationMetadataId": "3d85b034-e5ce-45f0-af46-b0247b745fa5" }
            },
            "Set_PageTitle_OneOff": {
              "type": "OpenApiConnection",
              "inputs": {
                "parameters": {
                  "notebookKey": "Meeting Notes|$|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Meeting Notes",
                  "sectionId": "@variables('varTargetSectionPagesUrl')",
                  "pageId": "@outputs('Compose_ConfirmedCreatedPageId_OneOff')",
                  "updates": [
                    { "target": "title", "action": "replace", "content": "@outputs('Compose_SafePageTitle_OneOff')" }
                  ]
                },
                "host": {
                  "apiId": "/providers/Microsoft.PowerApps/apis/shared_onenote",
                  "connection": "shared_onenote-1",
                  "operationId": "UpdatePageContent"
                }
              },
              "runAfter": { "Compose_ConfirmedCreatedPageId_OneOff": ["Succeeded"] },
              "metadata": { "operationMetadataId": "5f51baca-e9bb-45ad-ac9c-c9297fc8f9f6" }
            },
            "Delay_Post_Page_Creation_OneOff": {
              "type": "Wait",
              "inputs": { "interval": { "count": 5, "unit": "Second" } },
              "runAfter": { "Create_Page_OneOff": ["Succeeded"] },
              "metadata": { "operationMetadataId": "2dcdff12-4f5f-46ee-b928-17347e613ac5" }
            },
            "Compose_SafeSectionName_D2": {
              "type": "Compose",
              "inputs": "@if(empty(trim(coalesce(triggerBody()?['text_1'], ''))), 'Mtg - Untitled Meeting', concat('Mtg - ', substring(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(triggerBody()?['text_1'], '/', '-'), ':', '-'), '&', 'and'), '?', ''), '*', ''), '<', ''), '>', ''), '\"', ''), '|', ''), '#', ''), '''', ''), '%', ''), '~', ''), 0, min(43, length(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(triggerBody()?['text_1'], '/', '-'), ':', '-'), '&', 'and'), '?', ''), '*', ''), '<', ''), '>', ''), '\"', ''), '|', ''), '#', ''), '''', ''), '%', ''), '~', ''))))))",
              "runAfter": { "Compose_SafePageTitle_OneOff": ["Succeeded"] }
            },
            "Filter_OneNote_Section": {
              "type": "Query",
              "inputs": {
                "from": "@outputs('Get_Sections_D2')?['body/value']",
                "where": "@equals(item()?['name'], outputs('Compose_SafeSectionName'))"
              },
              "runAfter": { "Get_Sections_D2": ["Succeeded"] }
            },
            "Compose_SectionMatchCount_D2": {
              "type": "Compose",
              "inputs": "@string(length(body('Filter_OneNote_Section')))",
              "runAfter": { "Filter_OneNote_Section": ["Succeeded"] }
            },
            "Condition_Section_Exists_D2": {
              "type": "If",
              "expression": {
                "and": [
                  { "equals": [ "@true", "@true" ] }
                ]
              },
              "actions": {
                "For_each_D2": {
                  "type": "Foreach",
                  "foreach": "@body('Filter_OneNote_Section')",
                  "actions": {
                    "Set_varTargetSectionPagesUrl_D2_Exists": {
                      "type": "SetVariable",
                      "inputs": { "name": "varTargetSectionPagesUrl", "value": "@items('For_each_D2')?['pagesUrl']" }
                    },
                    "Set_varOneNoteResolverResult_Exists_D2": {
                      "type": "SetVariable",
                      "inputs": { "name": "varOneNoteResolverResult", "value": "ExistingSection" },
                      "runAfter": { "Set_varTargetSectionPagesUrl_D2_Exists": ["Succeeded"] }
                    }
                  }
                }
              },
              "else": {
                "actions": {
                  "Create_Section_D2": {
                    "type": "OpenApiConnection",
                    "inputs": {
                      "parameters": {
                        "body/name": "@outputs('Compose_SafeSectionName')",
                        "notebookKey": "Meeting Notes|$|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Meeting Notes"
                      },
                      "host": {
                        "apiId": "/providers/Microsoft.PowerApps/apis/shared_onenote",
                        "connection": "shared_onenote-1",
                        "operationId": "CreateSectionInNotebook"
                      }
                    }
                  },
                  "Set_varTargetSectionPagesUrl_D2_Created": {
                    "type": "SetVariable",
                    "inputs": { "name": "varTargetSectionPagesUrl", "value": "@outputs('Create_Section_D2')?['body']?['pagesUrl']" },
                    "runAfter": { "Create_Section_D2": ["Succeeded"] }
                  },
                  "Set_varOneNoteResolverResult_Created_D2": {
                    "type": "SetVariable",
                    "inputs": { "name": "varOneNoteResolverResult", "value": "CreatedSection" },
                    "runAfter": { "Set_varTargetSectionPagesUrl_D2_Created": ["Succeeded"] }
                  }
                }
              },
              "runAfter": { "Compose_Section_Match_Count_D2": ["Succeeded"] }
            },
            "Get_Sections_D2": {
              "type": "OpenApiConnection",
              "inputs": {
                "parameters": {
                  "notebookKey": "Meeting Notes|$|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Meeting Notes"
                },
                "host": {
                  "apiId": "/providers/Microsoft.PowerApps/apis/shared_onenote",
                  "connection": "shared_onenote",
                  "operationId": "GetSectionsInNotebook"
                }
              },
              "metadata": { "operationMetadataId": "9f34520a-1871-4a49-a436-bc5fba160b68" }
            },
            "Compose_Section_Match_Count_D2": {
              "type": "Compose",
              "inputs": "@length(body('Filter_OneNote_Section'))",
              "runAfter": { "Compose_SectionMatchCount_D2": ["Succeeded"] }
            }
          }
        },
        "runAfter": { "Compose_ExistingPageId": ["Succeeded"] },
        "metadata": { "operationMetadataId": "02e22bf0-4db1-4c31-8e9a-3e4d1ac0dc89" }
      }
    }
  },
  "runAfter": { "Condition_Mapping_Exists": ["Succeeded"] },
  "metadata": { "operationMetadataId": "a4c109f1-b7e0-40c9-83aa-b415aa217050" }
}
```

## Compose_AgentResponseSummary

```json
{
  "type": "Compose",
  "inputs": "@if(\n  equals(variables('varPageAction'), 'Created'),\n  concat('Created a new OneNote meeting page for \"', triggerBody()?['text_1'], '\".'),\n  if(\n    equals(variables('varPageAction'), 'UpdatedAppend'),\n    concat('Updated the existing OneNote meeting page for \"', triggerBody()?['text_1'], '\" by appending a safe automated update block.'),\n    if(\n      equals(variables('varPageAction'), 'ExistsNoCreate'),\n      concat('An existing OneNote meeting page was found for \"', triggerBody()?['text_1'], '\" and is ready for update.'),\n      concat('Processed OneNote meeting page request for \"', triggerBody()?['text_1'], '\".')\n    )\n  )\n)",
  "runAfter": { "Condition_Should_Create_Page": ["Succeeded"] },
  "metadata": { "operationMetadataId": "91031544-8d22-4c50-b14b-1be46e3147bd" }
}
```

## Compose_SP_Item_Count

```json
{
  "type": "Compose",
  "inputs": "@length(body('Get_items')?['value'])",
  "runAfter": { "Compose_AgentResponseSummary": ["Succeeded"] },
  "metadata": { "operationMetadataId": "1161e3dd-72b0-415c-bca0-278b64bdbb86" }
}
```

## Set_varOutStatus (as captured)

```json
{
  "type": "SetVariable",
  "inputs": {
    "name": "varOutStatus",
    "value": "@if(and(contains(createArray('Created','Updated','UpdatedAppend'), variables('varPageAction')), equals(coalesce(outputs('Compose_MappingWriteSucceeded'), outputs('Compose_MappingWriteSucceeded_OneOff'), 'true'), 'true')), 'SUCCESS', if(and(contains(createArray('Created','Updated','UpdatedAppend'), variables('varPageAction')), equals(coalesce(outputs('Compose_MappingWriteSucceeded'), outputs('Compose_MappingWriteSucceeded_OneOff'), 'true'), 'false')), 'PARTIAL_SUCCESS', if(and(equals(toLower(string(triggerBody()?['text'])), 'true'), empty(variables('varOneNoteResolverResult'))), 'RECURRING_SETUP_REQUIRED', if(empty(variables('varTargetSectionPagesUrl')), 'SETUP_SECTION_NOT_FOUND', if(or(greater(int(coalesce(outputs('Compose_SectionMatchCount_Recurring'), '0')), 1), greater(int(coalesce(outputs('Compose_SectionMatchCount_OneOff'), '0')), 1)), 'SETUP_SECTION_AMBIGUOUS', if(and(empty(variables('varPageAction')), contains(createArray('ExistingMapping','ExistingSection'), variables('varOneNoteResolverResult'))), 'STALE_MAPPING', 'ERROR'))))))"
  },
  "runAfter": { "Compose_SP_Item_Count": ["Succeeded"] },
  "metadata": { "operationMetadataId": "d899c972-cc80-49d5-a3af-79127f6f608b" }
}
```

## Respond to the agent

```json
{
  "type": "Response",
  "kind": "Skills",
  "inputs": {
    "schema": {
      "type": "object",
      "properties": {
        "outisrecurring": { "title": "OutIsRecurring", "x-ms-dynamically-added": true, "type": "string" },
        "outmeetingtitle": { "title": "OutMeetingTitle", "x-ms-dynamically-added": true, "type": "string" },
        "outseriesmasterid": { "title": "OutSeriesMasterId", "x-ms-dynamically-added": true, "type": "string" },
        "outpagehtml": { "title": "OutPageHtml", "x-ms-dynamically-added": true, "type": "string" },
        "outspitemcount": { "title": "OutSPItemCount", "x-ms-dynamically-added": true, "type": "string" },
        "outmatchcount": { "title": "OutMatchCount", "x-ms-dynamically-added": true, "type": "string" },
        "outbranchresult": { "title": "OutBranchResult", "x-ms-dynamically-added": true, "type": "string" },
        "outonenoteresolverresult": { "title": "OutOneNoteResolverResult", "x-ms-dynamically-added": true, "type": "string" },
        "outtargetsectionpagesurl": { "title": "OutTargetSectionPagesUrl", "x-ms-dynamically-added": true, "type": "string" },
        "outcreatedpagelink": { "title": "OutCreatedPageLink", "x-ms-dynamically-added": true, "type": "string" },
        "outcreatedpageselfurl": { "title": "OutCreatedPageSelfUrl", "x-ms-dynamically-added": true, "type": "string" },
        "outfinaltargetsectionpagesurl": { "title": "OutFinalTargetSectionPagesUrl", "x-ms-dynamically-added": true, "type": "string" },
        "outresolverresult": { "title": "OutResolverResult", "x-ms-dynamically-added": true, "type": "string" },
        "outexistingpageselfurl": { "title": "OutExistingPageSelfUrl", "x-ms-dynamically-added": true, "type": "string" },
        "outpagedecision": { "title": "OutPageDecision", "x-ms-dynamically-added": true, "type": "string" },
        "outpageroute": { "title": "OutPageRoute", "x-ms-dynamically-added": true, "type": "string" },
        "outpageaction": { "title": "OutPageAction", "x-ms-dynamically-added": true, "type": "string" },
        "outupdatehtmlfragment": { "title": "OutUpdateHtmlFragment", "x-ms-dynamically-added": true, "type": "string" },
        "outagentresponsesummary": { "title": "OutAgentResponseSummary", "x-ms-dynamically-added": true, "type": "string" },
        "outstatus": { "title": "OutStatus", "x-ms-dynamically-added": true, "type": "string" }
      },
      "additionalProperties": {}
    },
    "statusCode": 200,
    "body": {
      "outisrecurring": "@{triggerBody()?['text']}",
      "outmeetingtitle": "@{triggerBody()?['text_1']}",
      "outseriesmasterid": "@{triggerBody()?['text_2']}",
      "outpagehtml": "@{triggerBody()?['text_3']}",
      "outspitemcount": "@{int(coalesce(outputs('Compose_SP_Item_Count'), 0))}",
      "outmatchcount": "@{variables('varFinalMatchCount')}",
      "outbranchresult": "@{variables('varFinalMatchCount')}",
      "outonenoteresolverresult": "@{variables('varOneNoteResolverResult')}",
      "outtargetsectionpagesurl": "@{variables('varTargetSectionPagesUrl')}",
      "outcreatedpagelink": "@{variables('varOutputPageLink')}",
      "outcreatedpageselfurl": "@{variables('varOutputPageSelfUrl')}",
      "outfinaltargetsectionpagesurl": "@{variables('varTargetSectionPagesUrl')}",
      "outresolverresult": "@{variables('varOneNoteResolverResult')}",
      "outexistingpageselfurl": "@{variables('varFinalExistingPageSelfUrl')}",
      "outpagedecision": "@{variables('varFinalPageDecision')}",
      "outpageroute": "@{equals(variables('varFinalPageDecision'), 'PAGE_EXISTS')}",
      "outpageaction": "@{variables('varPageAction')}",
      "outupdatehtmlfragment": "@{outputs('Compose_UpdateHtmlFragment')}",
      "outagentresponsesummary": "@{outputs('Compose_AgentResponseSummary')}",
      "outstatus": "@{variables('varOutStatus')}"
    }
  },
  "runAfter": { "Set_varOutStatus": ["Succeeded"] },
  "metadata": { "operationMetadataId": "8196f660-bf45-4346-b545-bb79f815e91c" }
}
```

---

*Snapshot only — no analysis. Findings from the end-to-end review will be filed separately.*
