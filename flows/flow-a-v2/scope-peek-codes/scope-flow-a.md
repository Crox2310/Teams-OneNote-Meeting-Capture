# Flow A v2 — Scope_FlowA Peek Code

**Confirmed green:** 26 September 2026
**Build status:** Complete and published

```json
{
  "type": "Scope",
  "actions": {
    "Scope_CalendarFetch": {
      "type": "Scope",
      "actions": {
        "CA01_Compose_DateContext": {
          "type": "Compose",
          "inputs": "@coalesce(triggerBody()?['text_1'], utcNow())"
        },
        "CA02_Compose_StartOfDay": {
          "type": "Compose",
          "inputs": "@formatDateTime(if(empty(trim(coalesce(outputs('CA01_Compose_DateContext'), ''))), utcNow(), outputs('CA01_Compose_DateContext')), 'yyyy-MM-ddT00:00:00Z')",
          "runAfter": { "CA01_Compose_DateContext": ["Succeeded"] }
        },
        "CA03_Compose_EndOfDay": {
          "type": "Compose",
          "inputs": "@formatDateTime(if(empty(trim(coalesce(outputs('CA01_Compose_DateContext'), ''))), utcNow(), outputs('CA01_Compose_DateContext')), 'yyyy-MM-ddT23:59:59Z')",
          "runAfter": { "CA02_Compose_StartOfDay": ["Succeeded"] }
        },
        "CA04_Get_Calendar_Events": {
          "type": "OpenApiConnection",
          "inputs": {
            "parameters": {
              "calendarId": "AAMkAGY0OGU4Mzk5LWQ4NTYtNDU4MS1hY2YyLTQxOWYwZjhiMWM1ZQBGAAAAAADWkXK1vW2mQ4SwNGpyD7SzBwB8mPnOPkRmT5-MxNoNopoPAAAAAAEGAAB8mPnOPkRmT5-MxNoNopoPAACPZPifAAA=",
              "startDateTimeUtc": "@outputs('CA02_Compose_StartOfDay')",
              "endDateTimeUtc": "@outputs('CA03_Compose_EndOfDay')"
            },
            "host": { "apiId": "/providers/Microsoft.PowerApps/apis/shared_office365", "connection": "shared_office365", "operationId": "GetEventsCalendarViewV3" }
          },
          "runAfter": { "CA03_Compose_EndOfDay": ["Succeeded"] }
        },
        "CA05_Compose_Raw_Candidates": {
          "type": "Compose",
          "inputs": "@body('CA04_Get_Calendar_Events')?['value']",
          "runAfter": { "CA04_Get_Calendar_Events": ["Succeeded"] }
        },
        "CA06_Filter_Exclude_Leave_And_Periods": {
          "type": "Query",
          "inputs": {
            "from": "@outputs('CA05_Compose_Raw_Candidates')",
            "where": "@not(or(contains(toLower(coalesce(item()?['subject'],'')), 'holiday'), contains(toLower(coalesce(item()?['subject'],'')), 'leave'), contains(toLower(coalesce(item()?['subject'],'')), 'a-l'), contains(toLower(coalesce(item()?['subject'],'')), 'ooo'), contains(toLower(coalesce(item()?['subject'],'')), 'out of office'), contains(toLower(coalesce(item()?['subject'],'')), 'bank holiday'), contains(toLower(coalesce(item()?['subject'],'')), 'smarter working'), contains(toLower(coalesce(item()?['subject'],'')), 'period reminder'), contains(toLower(coalesce(item()?['subject'],'')), 'manage email'), contains(toLower(coalesce(item()?['subject'],'')), 'quiet hour')))"
          },
          "runAfter": { "CA05_Compose_Raw_Candidates": ["Succeeded"] }
        },
        "CA07_Compose_Sorted_Candidates": {
          "type": "Compose",
          "inputs": "@sort(body('CA06_Filter_Exclude_Leave_And_Periods'), 'start')",
          "runAfter": { "CA06_Filter_Exclude_Leave_And_Periods": ["Succeeded"] }
        }
      }
    },
    "Scope_CandidateResolution": {
      "type": "Scope",
      "actions": {
        "CR01_Compose_Match_Count": {
          "type": "Compose",
          "inputs": "@length(outputs('CA07_Compose_Sorted_Candidates'))"
        },
        "CR02_Compose_Display_Date": {
          "type": "Compose",
          "inputs": "@formatDateTime(concat(outputs('CA01_Compose_DateContext'), 'T12:00:00'), 'ddd d MMM yyyy')",
          "runAfter": { "CR01_Compose_Match_Count": ["Succeeded"] }
        }
      },
      "runAfter": { "Scope_CalendarFetch": ["Succeeded"] }
    },
    "Scope_NoMatch": {
      "type": "Scope",
      "actions": {
        "NM01_Condition_Is_No_Match": {
          "type": "If",
          "expression": { "and": [{ "equals": ["@outputs('CR01_Compose_Match_Count')", 0] }] },
          "actions": {
            "NM02_Compose_No_Match_Status": { "type": "Compose", "inputs": "NO_MATCH" },
            "NM03_Compose_No_Match_Count": { "type": "Compose", "inputs": "@string(0)", "runAfter": { "NM02_Compose_No_Match_Status": ["Succeeded"] } },
            "NM04_Compose_No_Match_List": { "type": "Compose", "inputs": "@string('')", "runAfter": { "NM03_Compose_No_Match_Count": ["Succeeded"] } },
            "NM05_Compose_No_Match_Title": { "type": "Compose", "inputs": "@string('')", "runAfter": { "NM04_Compose_No_Match_List": ["Succeeded"] } },
            "NM06_Compose_No_Match_EventId": { "type": "Compose", "inputs": "@string('')", "runAfter": { "NM05_Compose_No_Match_Title": ["Succeeded"] } },
            "NM07_Compose_No_Match_IsRecurring": { "type": "Compose", "inputs": "@string('')", "runAfter": { "NM06_Compose_No_Match_EventId": ["Succeeded"] } },
            "NM08_Compose_No_Match_SeriesMasterId": { "type": "Compose", "inputs": "@string('')", "runAfter": { "NM07_Compose_No_Match_IsRecurring": ["Succeeded"] } },
            "NM09_Compose_No_Match_JoinUrl": { "type": "Compose", "inputs": "@string('')", "runAfter": { "NM08_Compose_No_Match_SeriesMasterId": ["Succeeded"] } },
            "NM10_Compose_No_Match_BodyPreview": { "type": "Compose", "inputs": "@string('')", "runAfter": { "NM09_Compose_No_Match_JoinUrl": ["Succeeded"] } },
            "NM11_Compose_No_Match_EndTime": { "type": "Compose", "inputs": "@string('')", "runAfter": { "NM10_Compose_No_Match_BodyPreview": ["Succeeded"] } }
          },
          "else": { "actions": {} }
        }
      },
      "runAfter": { "Scope_CandidateResolution": ["Succeeded"] }
    },
    "Scope_SingleMatch": {
      "type": "Scope",
      "actions": {
        "SM01_Condition_Is_Single_Match": {
          "type": "If",
          "expression": { "and": [{ "equals": ["@outputs('CR01_Compose_Match_Count')", 1] }] },
          "actions": {
            "SM02_Compose_Single_Event": { "type": "Compose", "inputs": "@outputs('CA07_Compose_Sorted_Candidates')[0]" },
            "SM03_Compose_Single_IsRecurring": { "type": "Compose", "inputs": "@if(empty(coalesce(outputs('SM02_Compose_Single_Event')?['seriesMasterId'], '')), 'false', 'true')", "runAfter": { "SM02_Compose_Single_Event": ["Succeeded"] } },
            "SM04_Compose_Single_SeriesMasterId": { "type": "Compose", "inputs": "@coalesce(outputs('SM02_Compose_Single_Event')?['seriesMasterId'], '')", "runAfter": { "SM03_Compose_Single_IsRecurring": ["Succeeded"] } },
            "SM05_Compose_Single_Title": { "type": "Compose", "inputs": "@coalesce(outputs('SM02_Compose_Single_Event')?['subject'], '')", "runAfter": { "SM04_Compose_Single_SeriesMasterId": ["Succeeded"] } },
            "SM06_Compose_Single_EventId": { "type": "Compose", "inputs": "@coalesce(outputs('SM02_Compose_Single_Event')?['id'], '')", "runAfter": { "SM05_Compose_Single_Title": ["Succeeded"] } },
            "SM07_Compose_Single_EndTime": { "type": "Compose", "inputs": "@coalesce(outputs('SM02_Compose_Single_Event')?['end'], '')", "runAfter": { "SM06_Compose_Single_EventId": ["Succeeded"] } },
            "SM08_Compose_Single_BodyPreview_Raw": { "type": "Compose", "inputs": "@coalesce(outputs('SM02_Compose_Single_Event')?['body'], '')", "runAfter": { "SM07_Compose_Single_EndTime": ["Succeeded"] } },
            "SM09_Compose_Single_BodyPreview_Stripped": { "type": "Compose", "inputs": "@if(equals(outputs('SM08_Compose_Single_BodyPreview_Raw'), ''), '', substring(outputs('SM08_Compose_Single_BodyPreview_Raw'), 0, max(indexOf(outputs('SM08_Compose_Single_BodyPreview_Raw'), '</body>'), 0)))", "runAfter": { "SM08_Compose_Single_BodyPreview_Raw": ["Succeeded"] } },
            "SM10_Compose_Single_JoinUrl_Anchor_Index": { "type": "Compose", "inputs": "@indexOf(coalesce(outputs('SM02_Compose_Single_Event')?['body'], ''), 'href=\"https://teams.microsoft.com/l/meetup-join/')", "runAfter": { "SM09_Compose_Single_BodyPreview_Stripped": ["Succeeded"] } },
            "SM11_Compose_Single_JoinUrl_From_Anchor": { "type": "Compose", "inputs": "@if(equals(outputs('SM10_Compose_Single_JoinUrl_Anchor_Index'), -1), '', substring(coalesce(outputs('SM02_Compose_Single_Event')?['body'], ''), add(outputs('SM10_Compose_Single_JoinUrl_Anchor_Index'), 6), sub(length(coalesce(outputs('SM02_Compose_Single_Event')?['body'], '')), add(outputs('SM10_Compose_Single_JoinUrl_Anchor_Index'), 6))))", "runAfter": { "SM10_Compose_Single_JoinUrl_Anchor_Index": ["Succeeded"] } },
            "SM12_Compose_Single_JoinUrl_Extracted": { "type": "Compose", "inputs": "@if(equals(outputs('SM11_Compose_Single_JoinUrl_From_Anchor'), ''), '', substring(outputs('SM11_Compose_Single_JoinUrl_From_Anchor'), 0, if(equals(indexOf(outputs('SM11_Compose_Single_JoinUrl_From_Anchor'), '\"'), -1), length(outputs('SM11_Compose_Single_JoinUrl_From_Anchor')), indexOf(outputs('SM11_Compose_Single_JoinUrl_From_Anchor'), '\"'))))", "runAfter": { "SM11_Compose_Single_JoinUrl_From_Anchor": ["Succeeded"] } },
            "SM13_Compose_Single_JoinUrl_Final": { "type": "Compose", "inputs": "@if(empty(coalesce(outputs('SM02_Compose_Single_Event')?['onlineMeeting']?['joinUrl'], '')), outputs('SM12_Compose_Single_JoinUrl_Extracted'), outputs('SM02_Compose_Single_Event')?['onlineMeeting']?['joinUrl'])", "runAfter": { "SM12_Compose_Single_JoinUrl_Extracted": ["Succeeded"] } },
            "SM14_Compose_Single_Status": { "type": "Compose", "inputs": "MULTIPLE_MATCHES", "runAfter": { "SM13_Compose_Single_JoinUrl_Final": ["Succeeded"] } },
            "SM15_Compose_Single_MatchCount": { "type": "Compose", "inputs": "@string(1)", "runAfter": { "SM14_Compose_Single_Status": ["Succeeded"] } },
            "SM16_Compose_Single_CandidateList": { "type": "Compose", "inputs": "@string('')", "runAfter": { "SM15_Compose_Single_MatchCount": ["Succeeded"] } }
          },
          "else": { "actions": {} }
        }
      },
      "runAfter": { "Scope_NoMatch": ["Succeeded"] }
    },
    "Scope_MultiMatch": {
      "type": "Scope",
      "actions": {
        "MM01_Condition_Is_Multi_Match": {
          "type": "If",
          "expression": { "and": [{ "greater": ["@outputs('CR01_Compose_Match_Count')", 1] }] },
          "actions": {
            "MM02_Get_Mapping_Rows": {
              "type": "OpenApiConnection",
              "inputs": {
                "parameters": {
                  "dataset": "https://jsainsbury.sharepoint.com/sites/coplt",
                  "table": "186b3c9f-e758-4e85-83d5-685946614a0a",
                  "$filter": "OccurrenceDate eq '@{formatDateTime(outputs('CA01_Compose_DateContext'), 'yyyy-MM-dd')}'",
                  "$top": 50
                },
                "host": { "apiId": "/providers/Microsoft.PowerApps/apis/shared_sharepointonline", "connection": "shared_sharepointonline", "operationId": "GetItems" }
              }
            },
            "MM04_Build_Candidate_List_Loop": {
              "type": "Foreach",
              "foreach": "@outputs('CA07_Compose_Sorted_Candidates')",
              "actions": {
                "MM04a_Filter_Mapping_Match": { "type": "Query", "inputs": { "from": "@body('MM02_Get_Mapping_Rows')?['value']", "where": "@or(equals(item()?['SeriesMasterId'], coalesce(items('MM04_Build_Candidate_List_Loop')?['seriesMasterId'], '')), equals(item()?['MeetingId'], coalesce(items('MM04_Build_Candidate_List_Loop')?['id'], '')))" } },
                "MM04b_Compose_Status_Label": { "type": "Compose", "inputs": "@if(equals(coalesce(first(body('MM04a_Filter_Mapping_Match'))?['RecapCaptured'], false), true), ' - **Mtg Notes**', if(greater(length(body('MM04a_Filter_Mapping_Match')), 0), ' - **Captured**', ''))", "runAfter": { "MM04a_Filter_Mapping_Match": ["Succeeded"] } },
                "MM04c_Append_To_Candidate_List": { "type": "AppendToStringVariable", "inputs": { "name": "varCandidateListText", "value": "@concat(string(variables('varCandidateIndex')), '. ', coalesce(items('MM04_Build_Candidate_List_Loop')?['subject'], 'Untitled meeting'), outputs('MM04b_Compose_Status_Label'), decodeUriComponent('%0D%0A'))" }, "runAfter": { "MM04b_Compose_Status_Label": ["Succeeded"] } },
                "MM04d_Increment_Candidate_Index": { "type": "IncrementVariable", "inputs": { "name": "varCandidateIndex", "value": 1 }, "runAfter": { "MM04c_Append_To_Candidate_List": ["Succeeded"] } }
              },
              "runAfter": { "MM02_Get_Mapping_Rows": ["Succeeded"] }
            },
            "MM05_Compose_Multi_Status": { "type": "Compose", "inputs": "MULTIPLE_MATCHES", "runAfter": { "MM04_Build_Candidate_List_Loop": ["Succeeded"] } },
            "MM06_Compose_Multi_Match_Count": { "type": "Compose", "inputs": "@string(outputs('CR01_Compose_Match_Count'))", "runAfter": { "MM05_Compose_Multi_Status": ["Succeeded"] } },
            "MM07_Compose_Multi_Candidate_List": { "type": "Compose", "inputs": "@concat('Meetings for ', outputs('CR02_Compose_Display_Date'), decodeUriComponent('%0D%0A'), variables('varCandidateListText'))", "runAfter": { "MM06_Compose_Multi_Match_Count": ["Succeeded"] } },
            "MM08_Compose_Multi_Title": { "type": "Compose", "inputs": "@string('')", "runAfter": { "MM07_Compose_Multi_Candidate_List": ["Succeeded"] } },
            "MM09_Compose_Multi_EventId": { "type": "Compose", "inputs": "@string('')", "runAfter": { "MM08_Compose_Multi_Title": ["Succeeded"] } },
            "MM10_Compose_Multi_IsRecurring": { "type": "Compose", "inputs": "@string('')", "runAfter": { "MM09_Compose_Multi_EventId": ["Succeeded"] } },
            "MM11_Compose_Multi_SeriesMasterId": { "type": "Compose", "inputs": "@string('')", "runAfter": { "MM10_Compose_Multi_IsRecurring": ["Succeeded"] } },
            "MM12_Compose_Multi_JoinUrl": { "type": "Compose", "inputs": "@string('')", "runAfter": { "MM11_Compose_Multi_SeriesMasterId": ["Succeeded"] } },
            "MM13_Compose_Multi_EndTime": { "type": "Compose", "inputs": "@string('')", "runAfter": { "MM12_Compose_Multi_JoinUrl": ["Succeeded"] } }
          },
          "else": { "actions": {} }
        }
      },
      "runAfter": { "Scope_SingleMatch": ["Succeeded"] }
    },
    "Scope_Response": {
      "type": "Scope",
      "actions": {
        "RP01_Respond_To_Agent": {
          "type": "Response",
          "kind": "Skills",
          "inputs": {
            "statusCode": 200,
            "body": {
              "status": "@{coalesce(outputs('NM02_Compose_No_Match_Status'), outputs('SM14_Compose_Single_Status'), outputs('MM05_Compose_Multi_Status'), 'ERROR')}",
              "matchcount": "@{coalesce(outputs('NM03_Compose_No_Match_Count'), outputs('SM15_Compose_Single_MatchCount'), outputs('MM06_Compose_Multi_Match_Count'), '0')}",
              "candidatelist": "@{coalesce(outputs('NM04_Compose_No_Match_List'), outputs('SM16_Compose_Single_CandidateList'), outputs('MM07_Compose_Multi_Candidate_List'), '')}",
              "meetingtitle": "@{coalesce(outputs('NM05_Compose_No_Match_Title'), outputs('SM05_Compose_Single_Title'), outputs('MM08_Compose_Multi_Title'), '')}",
              "calendareventid": "@{coalesce(outputs('NM06_Compose_No_Match_EventId'), outputs('SM06_Compose_Single_EventId'), outputs('MM09_Compose_Multi_EventId'), '')}",
              "isrecurring": "@{coalesce(outputs('NM07_Compose_No_Match_IsRecurring'), outputs('SM03_Compose_Single_IsRecurring'), outputs('MM10_Compose_Multi_IsRecurring'), '')}",
              "seriesmasterid": "@{coalesce(outputs('NM08_Compose_No_Match_SeriesMasterId'), outputs('SM04_Compose_Single_SeriesMasterId'), outputs('MM11_Compose_Multi_SeriesMasterId'), '')}",
              "onlinemeetingurl": "@{coalesce(outputs('NM09_Compose_No_Match_JoinUrl'), outputs('SM13_Compose_Single_JoinUrl_Final'), outputs('MM12_Compose_Multi_JoinUrl'), '')}",
              "bodypreview": "@{coalesce(outputs('NM10_Compose_No_Match_BodyPreview'), outputs('SM09_Compose_Single_BodyPreview_Stripped'), '')}",
              "endtime": "@{coalesce(outputs('NM11_Compose_No_Match_EndTime'), outputs('SM07_Compose_Single_EndTime'), outputs('MM13_Compose_Multi_EndTime'), '')}"
            }
          }
        }
      },
      "runAfter": { "Scope_MultiMatch": ["Succeeded"] }
    }
  },
  "runAfter": { "TV02_Initialise_Candidate_Index": ["Succeeded"] }
}
```

## Top-level actions (outside Scope_FlowA)

### TV01 Initialise Candidate List Text
```json
{
  "type": "InitializeVariable",
  "inputs": { "variables": [{ "name": "varCandidateListText", "type": "string" }] },
  "runAfter": {}
}
```

### TV02 Initialise Candidate Index
```json
{
  "type": "InitializeVariable",
  "inputs": { "variables": [{ "name": "varCandidateIndex", "type": "integer", "value": 1 }] },
  "runAfter": { "TV01_Initialise_Candidate_List_Text": ["Succeeded"] }
}
```

## Platform learnings recorded during build

- `indexOf()` does not support arrays in this environment (confirmed 26 Sep 2026)
- `InitializeVariable` cannot be nested inside any Scope or Condition — must be top-level
- Dynamic Content picker inserts internal IDs not action names — always type expressions manually
- New Copilot Studio Designer assigns trigger input keys sequentially (text, text_1, text_2...)
