# Scope_FlowB_Parent — Confirmed Published Peek Code

**Flow:** PA - Meeting Capture - B - Router - v2
**Date confirmed:** 27 September 2026
**Status:** Published. RT03a (recurring child) absent — wired in when recurring child is built. RT03b correctly points at one-off child.

## Trigger

```json
{
  "type": "Request",
  "kind": "Skills",
  "inputs": {
    "schema": {
      "type": "object",
      "properties": {
        "text":   { "title": "IsRecurring",    "type": "string" },
        "text_1": { "title": "MeetingTitle",   "type": "string" },
        "text_2": { "title": "SeriesMasterId", "type": "string" },
        "text_3": { "title": "PageHtml",       "type": "string" },
        "text_4": { "title": "MeetingId",      "type": "string" },
        "text_5": { "title": "OccurrenceDate", "type": "string" },
        "text_6": { "title": "EndTime",        "type": "string" },
        "text_7": { "title": "JoinUrl",        "type": "string" }
      },
      "required": ["text", "text_1", "text_2", "text_3", "text_4"]
    }
  }
}
```

## Outer Scope

```json
{
  "type": "Scope",
  "actions": {
    "Scope_Router": {
      "type": "Scope",
      "actions": {
        "RT01_Compose_Route_Key": {
          "type": "Compose",
          "inputs": "@if(empty(triggerBody()?['text_2']), 'ONEOFF', 'RECURRING')"
        },
        "RT02_Condition_Route": {
          "type": "If",
          "expression": { "and": [{ "equals": ["@outputs('RT01_Compose_Route_Key')", "ONEOFF"] }] },
          "actions": {
            "RT03b_Run_OneOff_Child": {
              "type": "Workflow",
              "inputs": {
                "host": { "workflowReferenceName": "5bcc208f-3aba-f111-aaaf-002248a25cfd" },
                "body": {
                  "text":   "@triggerBody()?['text_1']",
                  "text_1": "@triggerBody()?['text_4']",
                  "text_2": "@triggerBody()?['text_5']",
                  "text_3": "@triggerBody()?['text_3']",
                  "text_4": "@triggerBody()?['text_6']",
                  "text_5": "@triggerBody()?['text_7']"
                }
              }
            }
          },
          "else": { "actions": {} },
          "runAfter": { "RT01_Compose_Route_Key": ["SUCCEEDED"] }
        }
      }
    },
    "Scope_Relay": {
      "type": "Scope",
      "actions": {
        "RL01_Compose_Out_IsRecurring":               { "type": "Compose", "inputs": "@triggerBody()?['text']" },
        "RL02_Compose_Out_MeetingTitle":              { "type": "Compose", "inputs": "@triggerBody()?['text_1']",  "runAfter": { "RL01_Compose_Out_IsRecurring": ["SUCCEEDED"] } },
        "RL03_Compose_Out_SeriesMasterId":            { "type": "Compose", "inputs": "@triggerBody()?['text_2']",  "runAfter": { "RL02_Compose_Out_MeetingTitle": ["SUCCEEDED"] } },
        "RL04_Compose_Out_PageHtml":                  { "type": "Compose", "inputs": "@triggerBody()?['text_3']",  "runAfter": { "RL03_Compose_Out_SeriesMasterId": ["SUCCEEDED"] } },
        "RL05_Compose_Out_SP_Item_Count":             { "type": "Compose", "inputs": "@coalesce(body('RT03a_Run_Recurring_Child')?['outspitemcount'], body('RT03b_Run_OneOff_Child')?['outspitemcount'], '')", "runAfter": { "RL04_Compose_Out_PageHtml": ["SUCCEEDED"] } },
        "RL06_Compose_Out_Match_Count":               { "type": "Compose", "inputs": "@coalesce(body('RT03a_Run_Recurring_Child')?['outmatchcount'], body('RT03b_Run_OneOff_Child')?['outmatchcount'], '')", "runAfter": { "RL05_Compose_Out_SP_Item_Count": ["SUCCEEDED"] } },
        "RL07_Compose_Out_Branch_Result":             { "type": "Compose", "inputs": "@coalesce(body('RT03a_Run_Recurring_Child')?['outbranchresult'], body('RT03b_Run_OneOff_Child')?['outbranchresult'], '')", "runAfter": { "RL06_Compose_Out_Match_Count": ["SUCCEEDED"] } },
        "RL08_Compose_Out_OneNote_Resolver_Result":   { "type": "Compose", "inputs": "@coalesce(body('RT03a_Run_Recurring_Child')?['outonenoteresolverresult'], body('RT03b_Run_OneOff_Child')?['outonenoteresolverresult'], '')", "runAfter": { "RL07_Compose_Out_Branch_Result": ["SUCCEEDED"] } },
        "RL09_Compose_Out_Target_Section_Pages_Url": { "type": "Compose", "inputs": "@coalesce(body('RT03a_Run_Recurring_Child')?['outtargetsectionpagesurl'], body('RT03b_Run_OneOff_Child')?['outtargetsectionpagesurl'], '')", "runAfter": { "RL08_Compose_Out_OneNote_Resolver_Result": ["SUCCEEDED"] } },
        "RL10_Compose_Out_Created_Page_Link":         { "type": "Compose", "inputs": "@coalesce(body('RT03a_Run_Recurring_Child')?['outcreatedpagelink'], body('RT03b_Run_OneOff_Child')?['outcreatedpagelink'], '')", "runAfter": { "RL09_Compose_Out_Target_Section_Pages_Url": ["SUCCEEDED"] } },
        "RL11_Compose_Out_Created_Page_Self_Url":     { "type": "Compose", "inputs": "@coalesce(body('RT03a_Run_Recurring_Child')?['outcreatedpageselfurl'], body('RT03b_Run_OneOff_Child')?['outcreatedpageselfurl'], '')", "runAfter": { "RL10_Compose_Out_Created_Page_Link": ["SUCCEEDED"] } },
        "RL12_Compose_Out_Final_Target_Section_Pages_Url": { "type": "Compose", "inputs": "@coalesce(body('RT03a_Run_Recurring_Child')?['outfinaltargetsectionpagesurl'], body('RT03b_Run_OneOff_Child')?['outfinaltargetsectionpagesurl'], '')", "runAfter": { "RL11_Compose_Out_Created_Page_Self_Url": ["SUCCEEDED"] } },
        "RL13_Compose_Out_Resolver_Result":           { "type": "Compose", "inputs": "@coalesce(body('RT03a_Run_Recurring_Child')?['outresolverresult'], body('RT03b_Run_OneOff_Child')?['outresolverresult'], '')", "runAfter": { "RL12_Compose_Out_Final_Target_Section_Pages_Url": ["SUCCEEDED"] } },
        "RL14_Compose_Out_Existing_Page_Self_Url":    { "type": "Compose", "inputs": "@coalesce(body('RT03a_Run_Recurring_Child')?['outexistingpageselfurl'], body('RT03b_Run_OneOff_Child')?['outexistingpageselfurl'], '')", "runAfter": { "RL13_Compose_Out_Resolver_Result": ["SUCCEEDED"] } },
        "RL15_Compose_Out_Page_Decision":             { "type": "Compose", "inputs": "@coalesce(body('RT03a_Run_Recurring_Child')?['outpagedecision'], body('RT03b_Run_OneOff_Child')?['outpagedecision'], '')", "runAfter": { "RL14_Compose_Out_Existing_Page_Self_Url": ["SUCCEEDED"] } },
        "RL16_Compose_Out_Page_Route":                { "type": "Compose", "inputs": "@coalesce(body('RT03a_Run_Recurring_Child')?['outpageroute'], body('RT03b_Run_OneOff_Child')?['outpageroute'], '')", "runAfter": { "RL15_Compose_Out_Page_Decision": ["SUCCEEDED"] } },
        "RL17_Compose_Out_Page_Action":               { "type": "Compose", "inputs": "@coalesce(body('RT03a_Run_Recurring_Child')?['outpageaction'], body('RT03b_Run_OneOff_Child')?['outpageaction'], '')", "runAfter": { "RL16_Compose_Out_Page_Route": ["SUCCEEDED"] } },
        "RL18_Compose_Out_Update_Html_Fragment":      { "type": "Compose", "inputs": "@coalesce(body('RT03a_Run_Recurring_Child')?['outupdatehtmlfragment'], body('RT03b_Run_OneOff_Child')?['outupdatehtmlfragment'], '')", "runAfter": { "RL17_Compose_Out_Page_Action": ["SUCCEEDED"] } },
        "RL19_Compose_Out_Agent_Response_Summary":    { "type": "Compose", "inputs": "@coalesce(body('RT03a_Run_Recurring_Child')?['outagentresponsesummary'], body('RT03b_Run_OneOff_Child')?['outagentresponsesummary'], '')", "runAfter": { "RL18_Compose_Out_Update_Html_Fragment": ["SUCCEEDED"] } },
        "RL20_Compose_Out_Status":                   { "type": "Compose", "inputs": "@coalesce(body('RT03a_Run_Recurring_Child')?['outstatus'], body('RT03b_Run_OneOff_Child')?['outstatus'], 'ERROR')", "runAfter": { "RL19_Compose_Out_Agent_Response_Summary": ["SUCCEEDED"] } },
        "RL_Respond": {
          "type": "Response",
          "kind": "Skills",
          "inputs": {
            "statusCode": 200,
            "body": {
              "outisrecurring":            "@{outputs('RL01_Compose_Out_IsRecurring')}",
              "outmeetingtitle":           "@{outputs('RL02_Compose_Out_MeetingTitle')}",
              "outseriesmasterid":         "@{outputs('RL03_Compose_Out_SeriesMasterId')}",
              "outpagehtml":               "@{outputs('RL04_Compose_Out_PageHtml')}",
              "outspitemcount":            "@{outputs('RL05_Compose_Out_SP_Item_Count')}",
              "outmatchcount":             "@{outputs('RL06_Compose_Out_Match_Count')}",
              "outbranchresult":           "@{outputs('RL07_Compose_Out_Branch_Result')}",
              "outonenoteresolverresult":  "@{outputs('RL08_Compose_Out_OneNote_Resolver_Result')}",
              "outtargetsectionpagesurl": "@{outputs('RL09_Compose_Out_Target_Section_Pages_Url')}",
              "outcreatedpagelink":        "@{outputs('RL10_Compose_Out_Created_Page_Link')}",
              "outcreatedpageselfurl":     "@{outputs('RL11_Compose_Out_Created_Page_Self_Url')}",
              "outfinaltargetsectionpagesurl": "@{outputs('RL12_Compose_Out_Final_Target_Section_Pages_Url')}",
              "outresolverresult":         "@{outputs('RL13_Compose_Out_Resolver_Result')}",
              "outexistingpageselfurl":    "@{outputs('RL14_Compose_Out_Existing_Page_Self_Url')}",
              "outpagedecision":           "@{outputs('RL15_Compose_Out_Page_Decision')}",
              "outpageroute":              "@{outputs('RL16_Compose_Out_Page_Route')}",
              "outpageaction":             "@{outputs('RL17_Compose_Out_Page_Action')}",
              "outupdatehtmlfragment":     "@{outputs('RL18_Compose_Out_Update_Html_Fragment')}",
              "outagentresponsesummary":   "@{outputs('RL19_Compose_Out_Agent_Response_Summary')}",
              "outstatus":                 "@{outputs('RL20_Compose_Out_Status')}"
            }
          },
          "runAfter": { "RL20_Compose_Out_Status": ["SUCCEEDED"] }
        }
      },
      "runAfter": { "Scope_Router": ["SUCCEEDED"] }
    }
  },
  "runAfter": {}
}
```

## To do when recurring child is built

- Add `RT03a_Run_Recurring_Child` to the False branch of RT02, pointing at the recurring child's flow ID
- Set `Scope_Relay` runAfter `Scope_Router` to `Succeeded AND Failed`
- Re-push this file with the updated Peek Code
