# Scope_FlowB_OneOff — Confirmed Green Peek Code

**Flow:** PA - Meeting Capture - B - OneOff - v2
**Date confirmed:** 27 September 2026
**Status:** Published, 0 errors, 1 warning (50-action advisory — ignorable, flow is called as child not from Power App directly)

## Trigger

```json
{
  "type": "Request",
  "kind": "PowerAppV2",
  "inputs": {
    "schema": {
      "type": "object",
      "properties": {
        "text": { "title": "MeetingTitle", "type": "string" },
        "text_1": { "title": "MeetingId", "type": "string" },
        "text_2": { "title": "OccurrenceDate", "type": "string" },
        "text_3": { "title": "PageHtml", "type": "string" },
        "text_4": { "title": "EndTime", "type": "string" },
        "text_5": { "title": "JoinUrl", "type": "string" }
      },
      "required": ["text", "text_1", "text_2", "text_3"]
    }
  }
}
```

## Outer Scope

```json
{
  "type": "Scope",
  "actions": {
    "Scope_Normalize": {
      "type": "Scope",
      "actions": {
        "NZ00a_Compose_MeetingTitle": { "type": "Compose", "inputs": "@triggerBody()?['text']" },
        "NZ00b_Compose_MeetingId": { "type": "Compose", "inputs": "@triggerBody()?['text_1']", "runAfter": { "NZ00a_Compose_MeetingTitle": ["SUCCEEDED"] } },
        "NZ00c_Compose_OccurrenceDate": { "type": "Compose", "inputs": "@triggerBody()?['text_2']", "runAfter": { "NZ00b_Compose_MeetingId": ["SUCCEEDED"] } },
        "NZ00d_Compose_PageHtml": { "type": "Compose", "inputs": "@triggerBody()?['text_3']", "runAfter": { "NZ00c_Compose_OccurrenceDate": ["SUCCEEDED"] } },
        "NZ00e_Compose_EndTime": { "type": "Compose", "inputs": "@triggerBody()?['text_4']", "runAfter": { "NZ00d_Compose_PageHtml": ["SUCCEEDED"] } },
        "NZ00f_Compose_JoinUrl": { "type": "Compose", "inputs": "@triggerBody()?['text_5']", "runAfter": { "NZ00e_Compose_EndTime": ["SUCCEEDED"] } },
        "NZ01_Compose_Safe_Page_Title": {
          "type": "Compose",
          "inputs": "@if(empty(trim(coalesce(outputs('NZ00a_Compose_MeetingTitle'), ''))), 'Untitled Meeting', concat(substring(replace(replace(replace(replace(outputs('NZ00a_Compose_MeetingTitle'), '&', 'and'), '<', ''), '>', ''), '\"', ''), 0, min(150, length(replace(replace(replace(replace(outputs('NZ00a_Compose_MeetingTitle'), '&', 'and'), '<', ''), '>', ''), '\"', '')))), if(empty(coalesce(outputs('NZ00c_Compose_OccurrenceDate'), '')), '', concat(' - ', formatDateTime(outputs('NZ00c_Compose_OccurrenceDate'), 'd MMM yyyy')))))",
          "runAfter": { "NZ00f_Compose_JoinUrl": ["SUCCEEDED"] }
        },
        "NZ02_Compose_Update_Html_Fragment": {
          "type": "Compose",
          "inputs": "@concat('<hr><h2>Automated update</h2><p><strong>Updated by:</strong> Meeting Capture Agent</p><p><strong>Update note:</strong> Meeting details were refreshed by the automation. Existing human-entered notes were preserved below.</p>', outputs('NZ00d_Compose_PageHtml'))",
          "runAfter": { "NZ01_Compose_Safe_Page_Title": ["SUCCEEDED"] }
        },
        "NZ03_Compose_Target_Section_Pages_Url": {
          "type": "Compose",
          "inputs": "https://www.onenote.com/api/v1.0/myOrganization/siteCollections/b5f8860c-4772-4e8b-b340-e80ba9d490fa/sites/d814850f-59bb-4182-92b7-e25d8c6a0487/notes/sections/1-cbb5e863-2bd7-458f-9944-4fec9d20607b/pages",
          "runAfter": { "NZ02_Compose_Update_Html_Fragment": ["SUCCEEDED"] }
        }
      }
    },
    "Scope_MappingLookup": {
      "type": "Scope",
      "actions": {
        "ML01_Get_items": {
          "type": "OpenApiConnection",
          "inputs": {
            "parameters": {
              "dataset": "https://jsainsbury.sharepoint.com/sites/coplt",
              "table": "186b3c9f-e758-4e85-83d5-685946614a0a",
              "$top": 500
            },
            "host": { "apiId": "/providers/Microsoft.PowerApps/apis/shared_sharepointonline", "connection": "shared_sharepointonline", "operationId": "GetItems" }
          }
        },
        "ML02_Filter_Existing_Mapping_OneOff": {
          "type": "Query",
          "inputs": {
            "from": "@body('ML01_Get_items')?['value']",
            "where": "@and(equals(item()?['MeetingId'], outputs('NZ00b_Compose_MeetingId')), equals(item()?['OccurrenceDate'], outputs('NZ00c_Compose_OccurrenceDate')))"
          },
          "runAfter": { "ML01_Get_items": ["SUCCEEDED"] }
        },
        "ML03_Compose_Match_Count": { "type": "Compose", "inputs": "@length(body('ML02_Filter_Existing_Mapping_OneOff'))", "runAfter": { "ML02_Filter_Existing_Mapping_OneOff": ["SUCCEEDED"] } },
        "ML04_Compose_Mapping_Exists": { "type": "Compose", "inputs": "@greater(outputs('ML03_Compose_Match_Count'), 0)", "runAfter": { "ML03_Compose_Match_Count": ["SUCCEEDED"] } }
      },
      "runAfter": { "Scope_Normalize": ["SUCCEEDED"] }
    },
    "Condition_Mapping_Exists": {
      "type": "If",
      "expression": { "and": [{ "equals": ["@outputs('ML04_Compose_Mapping_Exists')", true] }] },
      "actions": {
        "Scope_ExistingMapping": {
          "type": "Scope",
          "actions": {
            "EM01_Compose_Existing_Page_Self_Url": { "type": "Compose", "inputs": "@coalesce(first(body('ML02_Filter_Existing_Mapping_OneOff'))?['PageSelfUrl'], '')" },
            "EM02_Compose_Existing_Page_Web_Url": { "type": "Compose", "inputs": "@coalesce(first(body('ML02_Filter_Existing_Mapping_OneOff'))?['PageWebUrl'], '')", "runAfter": { "EM01_Compose_Existing_Page_Self_Url": ["SUCCEEDED"] } },
            "EM03_Compose_Existing_Row_Id": { "type": "Compose", "inputs": "@string(first(body('ML02_Filter_Existing_Mapping_OneOff'))?['ID'])", "runAfter": { "EM02_Compose_Existing_Page_Web_Url": ["SUCCEEDED"] } }
          }
        }
      },
      "else": {
        "actions": {
          "Scope_NewMapping": {
            "type": "Scope",
            "actions": {
              "NM01_Create_Mapping_Item_OneOff": {
                "type": "OpenApiConnection",
                "inputs": {
                  "parameters": {
                    "dataset": "https://jsainsbury.sharepoint.com/sites/coplt",
                    "table": "186b3c9f-e758-4e85-83d5-685946614a0a",
                    "item/Title": "Mapping",
                    "item/MeetingTitle": "@outputs('NZ00a_Compose_MeetingTitle')",
                    "item/SectionPagesUrl": "@outputs('NZ03_Compose_Target_Section_Pages_Url')",
                    "item/Status/Value": "Active",
                    "item/MeetingId": "@outputs('NZ00b_Compose_MeetingId')",
                    "item/OccurrenceDate": "@outputs('NZ00c_Compose_OccurrenceDate')",
                    "item/JoinUrl": "@trim(coalesce(outputs('NZ00f_Compose_JoinUrl'), ''))",
                    "item/ChatCaptured": false,
                    "item/EndTime": "@coalesce(outputs('NZ00e_Compose_EndTime'), '')",
                    "item/RecapCaptured": false
                  },
                  "host": { "apiId": "/providers/Microsoft.PowerApps/apis/shared_sharepointonline", "connection": "shared_sharepointonline", "operationId": "PostItem" }
                }
              },
              "NM02_Compose_New_Row_Id": { "type": "Compose", "inputs": "@string(outputs('NM01_Create_Mapping_Item_OneOff')?['body/ID'])", "runAfter": { "NM01_Create_Mapping_Item_OneOff": ["SUCCEEDED"] } },
              "NM03_Compose_Mapping_Write_Succeeded": { "type": "Compose", "inputs": "@if(equals(outputs('NM01_Create_Mapping_Item_OneOff')?['statusCode'], 201), 'true', 'false')", "runAfter": { "NM02_Compose_New_Row_Id": ["SUCCEEDED"] } }
            }
          }
        }
      },
      "runAfter": { "Scope_MappingLookup": ["SUCCEEDED"] }
    },
    "Scope_PageResolve": {
      "type": "Scope",
      "actions": {
        "PG01_Get_Pages_In_Section": {
          "type": "OpenApiConnection",
          "inputs": {
            "parameters": {
              "notebookKey": "Meeting Notes|$|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Meeting Notes",
              "sectionId": "@outputs('NZ03_Compose_Target_Section_Pages_Url')"
            },
            "host": { "apiId": "/providers/Microsoft.PowerApps/apis/shared_onenote", "connection": "shared_onenote", "operationId": "GetPagesInSection" }
          }
        },
        "PG02_Filter_Pages_By_Title": { "type": "Query", "inputs": { "from": "@outputs('PG01_Get_Pages_In_Section')?['body']?['value']", "where": "@contains(item()?['title'],formatDateTime(outputs('NZ00c_Compose_OccurrenceDate'), 'd MMM yyyy'))" }, "runAfter": { "PG01_Get_Pages_In_Section": ["SUCCEEDED"] } },
        "PG03_Compose_Page_Match_Count": { "type": "Compose", "inputs": "@length(body('PG02_Filter_Pages_By_Title'))", "runAfter": { "PG02_Filter_Pages_By_Title": ["SUCCEEDED"] } },
        "PG04_Compose_Page_Decision": { "type": "Compose", "inputs": "@if(greater(outputs('PG03_Compose_Page_Match_Count'), 0), 'PAGE_EXISTS', 'PAGE_NOT_FOUND')", "runAfter": { "PG03_Compose_Page_Match_Count": ["SUCCEEDED"] } },
        "PG05_Condition_Page_Exists": {
          "type": "If",
          "expression": { "and": [{ "equals": ["@outputs('PG04_Compose_Page_Decision')", "PAGE_EXISTS"] }] },
          "actions": {
            "PG06_Compose_Existing_Page_Id": { "type": "Compose", "inputs": "@if(greater(length(body('PG02_Filter_Pages_By_Title')), 0), first(body('PG02_Filter_Pages_By_Title'))?['id'], '')" },
            "PG07_Update_Page_Content": { "type": "OpenApiConnection", "inputs": { "parameters": { "notebookKey": "Meeting Notes|$|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Meeting Notes", "sectionId": "@outputs('NZ03_Compose_Target_Section_Pages_Url')", "pageId": "@outputs('PG06_Compose_Existing_Page_Id')", "updates": [{ "target": "body", "action": "append", "content": "@outputs('NZ02_Compose_Update_Html_Fragment')" }] }, "host": { "apiId": "/providers/Microsoft.PowerApps/apis/shared_onenote", "connection": "shared_onenote", "operationId": "UpdatePageContent" } }, "runAfter": { "PG06_Compose_Existing_Page_Id": ["SUCCEEDED"] } },
            "PG08_Compose_Page_Action": { "type": "Compose", "inputs": "UpdatedAppend", "runAfter": { "PG07_Update_Page_Content": ["SUCCEEDED"] } },
            "PG09_Compose_Created_Page_Link": { "type": "Compose", "inputs": "@coalesce(outputs('EM02_Compose_Existing_Page_Web_Url'), '')", "runAfter": { "PG08_Compose_Page_Action": ["SUCCEEDED"] } },
            "PG10_Compose_Created_Page_Self_Url": { "type": "Compose", "inputs": "@coalesce(outputs('EM01_Compose_Existing_Page_Self_Url'), '')", "runAfter": { "PG09_Compose_Created_Page_Link": ["SUCCEEDED"] } }
          },
          "else": {
            "actions": {
              "PG11_Create_Page_In_Section": { "type": "OpenApiConnection", "inputs": { "parameters": { "notebookKey": "Meeting Notes|$|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Meeting Notes", "sectionId": "@outputs('NZ03_Compose_Target_Section_Pages_Url')", "pageContent": "<p class=\"editor-paragraph\">@{outputs('NZ00d_Compose_PageHtml')}</p>" }, "host": { "apiId": "/providers/Microsoft.PowerApps/apis/shared_onenote", "connection": "shared_onenote", "operationId": "CreatePageInSection" } } },
              "PG12_Compose_Page_Action": { "type": "Compose", "inputs": "Created", "runAfter": { "PG11_Create_Page_In_Section": ["SUCCEEDED"] } },
              "PG13_Compose_Created_Page_Link": { "type": "Compose", "inputs": "@coalesce(outputs('PG11_Create_Page_In_Section')?['body']?['links']?['oneNoteWebUrl']?['href'], '')", "runAfter": { "PG12_Compose_Page_Action": ["SUCCEEDED"] } },
              "PG14_Compose_Created_Page_Self_Url": { "type": "Compose", "inputs": "@coalesce(outputs('PG11_Create_Page_In_Section')?['body']?['self'], '')", "runAfter": { "PG13_Compose_Created_Page_Link": ["SUCCEEDED"] } }
            }
          },
          "runAfter": { "PG04_Compose_Page_Decision": ["SUCCEEDED"] }
        },
        "PG15_Compose_Page_Action_Final": { "type": "Compose", "inputs": "@coalesce(outputs('PG08_Compose_Page_Action'), outputs('PG12_Compose_Page_Action'), '')", "runAfter": { "PG05_Condition_Page_Exists": ["SUCCEEDED"] } },
        "PG16_Compose_Page_Link_Final": { "type": "Compose", "inputs": "@coalesce(outputs('PG09_Compose_Created_Page_Link'), outputs('PG13_Compose_Created_Page_Link'), '')", "runAfter": { "PG15_Compose_Page_Action_Final": ["SUCCEEDED"] } },
        "PG17_Compose_Page_Self_Url_Final": { "type": "Compose", "inputs": "@coalesce(outputs('PG10_Compose_Created_Page_Self_Url'), outputs('PG14_Compose_Created_Page_Self_Url'), '')", "runAfter": { "PG16_Compose_Page_Link_Final": ["SUCCEEDED"] } }
      },
      "runAfter": { "Condition_Mapping_Exists": ["SUCCEEDED"] }
    },
    "Scope_WriteBack": {
      "type": "Scope",
      "actions": {
        "WB00_Compose_Target_Row_Id": { "type": "Compose", "inputs": "@coalesce(outputs('NM02_Compose_New_Row_Id'), outputs('EM03_Compose_Existing_Row_Id'), '')" },
        "WB01_Condition_Is_New_Page": {
          "type": "If",
          "expression": { "and": [{ "equals": ["@outputs('PG15_Compose_Page_Action_Final')", "Created"] }] },
          "actions": {
            "WB02_HTTP_Update_SP_Page_Urls": {
              "type": "OpenApiConnection",
              "inputs": {
                "parameters": {
                  "Uri": "@concat('https://jsainsbury.sharepoint.com/sites/coplt/_api/lists(''186b3c9f-e758-4e85-83d5-685946614a0a'')/items(', outputs('WB00_Compose_Target_Row_Id'), ')')",
                  "Method": "PATCH",
                  "CustomHeader1": "Content-Type = application/json;odata=verbose",
                  "CustomHeader3": "IF-MATCH = *",
                  "CustomHeader4": "X-HTTP-Method = MERGE",
                  "Body": "@concat('{\"__metadata\":{\"type\":\"SP.Data.RecurringMeetingSectionMapListItem\"},\"PageSelfUrl\":\"', outputs('PG17_Compose_Page_Self_Url_Final'), '\",\"PageWebUrl\":\"', outputs('PG16_Compose_Page_Link_Final'), '\"}')",
                  "ContentType": "application/json"
                },
                "host": { "apiId": "/providers/Microsoft.PowerApps/apis/shared_office365", "connection": "shared_office365", "operationId": "HttpRequest" }
              }
            }
          },
          "else": { "actions": {} },
          "runAfter": { "WB00_Compose_Target_Row_Id": ["SUCCEEDED"] }
        }
      },
      "runAfter": { "Scope_PageResolve": ["SUCCEEDED"] }
    },
    "Scope_Status": {
      "type": "Scope",
      "actions": {
        "ST01_Compose_Out_Status": { "type": "Compose", "inputs": "@if(and(contains(createArray('Created','Updated','UpdatedAppend'), outputs('PG15_Compose_Page_Action_Final')), equals(coalesce(outputs('NM03_Compose_Mapping_Write_Succeeded'), 'true'), 'true')), 'SUCCESS', if(and(contains(createArray('Created','Updated','UpdatedAppend'), outputs('PG15_Compose_Page_Action_Final')), equals(coalesce(outputs('NM03_Compose_Mapping_Write_Succeeded'), 'true'), 'false')), 'PARTIAL_SUCCESS', 'ERROR'))" },
        "ST02_Compose_Agent_Response_Summary": { "type": "Compose", "inputs": "@if(equals(outputs('PG15_Compose_Page_Action_Final'), 'Created'), concat('Created a new OneNote meeting page for \"', outputs('NZ00a_Compose_MeetingTitle'), '\".'), if(equals(outputs('PG15_Compose_Page_Action_Final'), 'UpdatedAppend'), concat('Updated the existing OneNote meeting page for \"', outputs('NZ00a_Compose_MeetingTitle'), '\" by appending a safe automated update block.'), if(equals(outputs('PG15_Compose_Page_Action_Final'), 'ExistsNoCreate'), concat('An existing OneNote meeting page was found for \"', outputs('NZ00a_Compose_MeetingTitle'), '\" and is ready for update.'), concat('Processed OneNote meeting page request for \"', outputs('NZ00a_Compose_MeetingTitle'), '\".'))))", "runAfter": { "ST01_Compose_Out_Status": ["SUCCEEDED"] } }
      },
      "runAfter": { "Scope_WriteBack": ["SUCCEEDED"] }
    },
    "Scope_Respond": {
      "type": "Scope",
      "actions": {
        "RP01_Compose_outspitemcount": { "type": "Compose", "inputs": "@int(coalesce(outputs('ML03_Compose_Match_Count'), 0))" },
        "RP02_Compose_outmatchcount": { "type": "Compose", "inputs": "@string(outputs('ML03_Compose_Match_Count'))", "runAfter": { "RP01_Compose_outspitemcount": ["SUCCEEDED"] } },
        "RP03_Compose_outbranchresult": { "type": "Compose", "inputs": "@string(outputs('ML03_Compose_Match_Count'))", "runAfter": { "RP02_Compose_outmatchcount": ["SUCCEEDED"] } },
        "RP04_Compose_outonenoteresolverresult": { "type": "Compose", "inputs": "@string('')", "runAfter": { "RP03_Compose_outbranchresult": ["SUCCEEDED"] } },
        "RP05_Compose_outtargetsectionpagesurl": { "type": "Compose", "inputs": "@outputs('NZ03_Compose_Target_Section_Pages_Url')", "runAfter": { "RP04_Compose_outonenoteresolverresult": ["SUCCEEDED"] } },
        "RP06_Compose_outcreatedpagelink": { "type": "Compose", "inputs": "@outputs('PG16_Compose_Page_Link_Final')", "runAfter": { "RP05_Compose_outtargetsectionpagesurl": ["SUCCEEDED"] } },
        "RP07_Compose_outcreatedpageselfurl": { "type": "Compose", "inputs": "@outputs('PG17_Compose_Page_Self_Url_Final')", "runAfter": { "RP06_Compose_outcreatedpagelink": ["SUCCEEDED"] } },
        "RP08_Compose_outfinaltargetsectionpagesurl": { "type": "Compose", "inputs": "@outputs('NZ03_Compose_Target_Section_Pages_Url')", "runAfter": { "RP07_Compose_outcreatedpageselfurl": ["SUCCEEDED"] } },
        "RP09_Compose_outresolverresult": { "type": "Compose", "inputs": "@string('')", "runAfter": { "RP08_Compose_outfinaltargetsectionpagesurl": ["SUCCEEDED"] } },
        "RP10_Compose_outexistingpageselfurl": { "type": "Compose", "inputs": "@coalesce(outputs('EM01_Compose_Existing_Page_Self_Url'), '')", "runAfter": { "RP09_Compose_outresolverresult": ["SUCCEEDED"] } },
        "RP11_Compose_outpagedecision": { "type": "Compose", "inputs": "@outputs('PG04_Compose_Page_Decision')", "runAfter": { "RP10_Compose_outexistingpageselfurl": ["SUCCEEDED"] } },
        "RP12_Compose_outpageroute": { "type": "Compose", "inputs": "@string(equals(outputs('PG04_Compose_Page_Decision'), 'PAGE_EXISTS'))", "runAfter": { "RP11_Compose_outpagedecision": ["SUCCEEDED"] } },
        "RP13_Compose_outpageaction": { "type": "Compose", "inputs": "@outputs('PG15_Compose_Page_Action_Final')", "runAfter": { "RP12_Compose_outpageroute": ["SUCCEEDED"] } },
        "RP14_Compose_outupdatehtmlfragment": { "type": "Compose", "inputs": "@outputs('NZ02_Compose_Update_Html_Fragment')", "runAfter": { "RP13_Compose_outpageaction": ["SUCCEEDED"] } },
        "RP15_Compose_outagentresponsesummary": { "type": "Compose", "inputs": "@outputs('ST02_Compose_Agent_Response_Summary')", "runAfter": { "RP14_Compose_outupdatehtmlfragment": ["SUCCEEDED"] } },
        "RP16_Compose_outstatus": { "type": "Compose", "inputs": "@outputs('ST01_Compose_Out_Status')", "runAfter": { "RP15_Compose_outagentresponsesummary": ["SUCCEEDED"] } },
        "Respond_to_a_Power_App_or_flow": {
          "type": "Response",
          "kind": "PowerApp",
          "inputs": {
            "statusCode": 200,
            "body": {
              "outspitemcount": "@{outputs('RP01_Compose_outspitemcount')}",
              "outmatchcount": "@{outputs('RP02_Compose_outmatchcount')}",
              "outbranchresult": "@{outputs('RP03_Compose_outbranchresult')}",
              "outonenoteresolverresult": "@{outputs('RP04_Compose_outonenoteresolverresult')}",
              "outtargetsectionpagesurl": "@{outputs('RP05_Compose_outtargetsectionpagesurl')}",
              "outcreatedpagelink": "@{outputs('RP06_Compose_outcreatedpagelink')}",
              "outcreatedpageselfurl": "@{outputs('RP07_Compose_outcreatedpageselfurl')}",
              "outfinaltargetsectionpagesurl": "@{outputs('RP08_Compose_outfinaltargetsectionpagesurl')}",
              "outresolverresult": "@{outputs('RP09_Compose_outresolverresult')}",
              "outexistingpageselfurl": "@{outputs('RP10_Compose_outexistingpageselfurl')}",
              "outpagedecision": "@{outputs('RP11_Compose_outpagedecision')}",
              "outpageroute": "@{outputs('RP12_Compose_outpageroute')}",
              "outpageaction": "@{outputs('RP13_Compose_outpageaction')}",
              "outupdatehtmlfragment": "@{outputs('RP14_Compose_outupdatehtmlfragment')}",
              "outagentresponsesummary": "@{outputs('RP15_Compose_outagentresponsesummary')}",
              "outstatus": "@{outputs('RP16_Compose_outstatus')}"
            }
          },
          "runAfter": { "RP16_Compose_outstatus": ["SUCCEEDED"] }
        }
      },
      "runAfter": { "Scope_Status": ["SUCCEEDED"] }
    }
  },
  "runAfter": {}
}
```
