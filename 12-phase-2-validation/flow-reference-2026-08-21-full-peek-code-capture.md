# Flow reference — full Peek Code + Topic YAML capture, 21 August 2026 (evening)

**Purpose:** raw reference snapshot of Flow B and the Topic, captured during the session covered by `session-2026-08-21-fb04-build-and-getitems-mystery.md`. This supersedes `flow-reference-2026-08-20-full-peek-code-capture.md`, which predates FB-04, the `Get_items` investigation, and the corruption-recovery cycle earlier the same day.

**Status at time of capture:** Flow B and the Topic both published, including FB-01 through FB-04. This is the current live state as far as evidence in this session shows.

---

## Topic YAML (full export)

```yaml
kind: AdaptiveDialog
modelDescription: |-
  Captures meeting notes by finding a user's Outlook meeting, allowing the user to select from multiple meetings, and creating a OneNote page for the selected meeting.

  Users might say: capture meetings, capture meeting, meeting capture, capture meeting notes, create meeting notes, record meeting notes.
beginDialog:
  kind: OnRecognizedIntent
  id: main
  intent:
    triggerQueries:
      - capture meeting notes
      - capture notes for
      - record meeting notes
      - create meeting notes
      - capture

  actions:
    - kind: SetVariable
      id: setVariable_2VpZMo
      displayName: C1_Set_DateContext
      variable: Topic.DateContext
      value: =Text(Today(), "yyyy-MM-dd")

    - kind: InvokeFlowAction
      id: invokeFlowAction_eBUGn8
      displayName: C2_Call_FlowA_Initial
      input:
        binding:
          text_1: NONE
          text_3: =Topic.DateContext

      output:
        binding:
          bodypreview: Topic.BodyPreview
          calendareventid: Topic.CalendarEventId
          candidatelist: Topic.CandidateList
          isrecurring: Topic.IsRecurring
          matchcount: Topic.MatchCount
          meetingtitle: Topic.MeetingTitle
          onlinemeetingurl: Topic.OnlineMeetingUrl
          seriesmasterid: Topic.SeriesMasterId
          status: Topic.Status

      flowId: d9d7ccf7-7d61-f111-a826-6045bde03856

    - kind: SetMultipleVariables
      id: setVariable_FXirQu
      assignments:
        - variable: Topic.MatchCount
          value: =Text(Topic.MatchCount)

        - variable: Topic.IsRecurring
          value: =Text(Topic.IsRecurring)

        - variable: Topic.MeetingTitle
          value: =Text(Topic.MeetingTitle)

        - variable: Topic.SeriesMasterId
          value: =Text(Topic.SeriesMasterId)

        - variable: Topic.CalendarEventId
          value: =Text(Topic.CalendarEventId)

        - variable: Topic.BodyPreview
          value: =Text(Topic.BodyPreview)

        - variable: Topic.OnlineMeetingUrl
          value: =Text(Topic.OnlineMeetingUrl)

    - kind: ConditionGroup
      id: conditionGroup_k0eXvc
      conditions:
        - id: conditionItem_oRVxZZ
          condition: =Topic.MatchCount = Text(0)
          displayName: C4_Check_MatchCount
          actions:
            - kind: SendActivity
              id: sendActivity_ZdhIF0
              displayName: C4A_NoMatch_Message (
              activity: I couldn't find any meetings for that day. Type P for previous day, N for next day, or a date (e.g. 28 Jun) to navigate.

      elseActions:
        - kind: SendActivity
          id: sendActivity_DKsGOT
          displayName: C5_Display_CandidateList
          activity: "{Topic.CandidateList}"

        - kind: Question
          id: question_XFJmje
          displayName: C6_Ask_SelectedNumber
          interruptionPolicy:
            allowInterruption: true

          variable: Topic.TopicSelectedNumber
          prompt: |-
            To change day select letter or enter date:
            - P for previous day
            - N for next day
            - Specific date (e.g. 23 Oct)
          entity:
            kind: StringPrebuiltEntity
            sensitivityLevel: None

        - kind: ConditionGroup
          id: conditionGroup_BsGPk1
          conditions:
            - id: conditionItem_2NuHzx
              condition: =Topic.TopicSelectedNumber = "P" || Topic.TopicSelectedNumber = "p"
              displayName: C6_Check_Input
              actions:
                - kind: SetVariable
                  id: setVariable_X0gXHk
                  variable: Topic.DateContext
                  value: =Text(DateAdd(DateValue(Topic.DateContext), -1, TimeUnit.Days), "yyyy-MM-dd")

                - kind: GotoAction
                  id: CHYopX
                  actionId: invokeFlowAction_eBUGn8

            - id: conditionItem_yERjqE
              condition: =Topic.TopicSelectedNumber = "N" || Topic.TopicSelectedNumber = "n"
              displayName: C6B_Check_N
              actions:
                - kind: SetVariable
                  id: setVariable_c7PbLR
                  variable: Topic.DateContext
                  value: =Text(DateAdd(DateValue(Topic.DateContext), 1, TimeUnit.Days), "yyyy-MM-dd")

                - kind: GotoAction
                  id: I8IQMw
                  actionId: invokeFlowAction_eBUGn8

            - id: conditionItem_Qz7mKp
              condition: =(!IsError(DateValue(Topic.TopicSelectedNumber)) && IsError(Value(Topic.TopicSelectedNumber))) || IsMatch(Topic.TopicSelectedNumber, "^\d{1,2}/\d{1,2}/\d{2,4}$")
              displayName: C6C_Check_Date
              actions:
                - kind: SetVariable
                  id: setVariable_Rk3nWx
                  variable: Topic.DateContext
                  value: |-
                    =Text(
                       If(
                         IsMatch(Topic.TopicSelectedNumber, "^\d{1,2}/\d{1,2}/\d{2,4}$"),
                         Date(
                           If(Len(Last(Split(Topic.TopicSelectedNumber, "/")).Value) <= 2, 2000 + Value(Last(Split(Topic.TopicSelectedNumber, "/")).Value), Value(Last(Split(Topic.TopicSelectedNumber, "/")).Value)),
                           Value(Index(Split(Topic.TopicSelectedNumber, "/"), 2).Value),
                           Value(First(Split(Topic.TopicSelectedNumber, "/")).Value)
                         ),
                         DateValue(Topic.TopicSelectedNumber)
                       ),
                       "yyyy-MM-dd")

                - kind: GotoAction
                  id: Tb9vLh
                  actionId: invokeFlowAction_eBUGn8

            - id: conditionItem_NumSelect
              condition: =!IsError(Value(Topic.TopicSelectedNumber))
              displayName: C6D_Check_Number
              actions:
                - kind: SetVariable
                  id: setVariable_C6D_noop
                  variable: Topic.DateContext
                  value: =Topic.DateContext

          elseActions:
            - kind: SendActivity
              id: sendActivity_C6D_Unrecognised
              displayName: C6D_Unrecognised_Input
              activity: Sorry, I didn't recognise that. Type P for previous day, N for next day, or a date like 23 Oct or 23/10/26.

            - kind: GotoAction
              id: goto_C6D_reprompt
              actionId: question_XFJmje

        - kind: InvokeFlowAction
          id: invokeFlowAction_bIIKPf
          input:
            binding:
              text_1: =Topic.TopicSelectedNumber
              text_3: =Topic.DateContext

          output:
            binding:
              bodypreview: Topic.BodyPreview
              calendareventid: Topic.CalendarEventId
              candidatelist: Topic.CandidateList
              isrecurring: Topic.IsRecurring
              matchcount: Topic.MatchCount
              meetingtitle: Topic.MeetingTitle
              onlinemeetingurl: Topic.OnlineMeetingUrl
              seriesmasterid: Topic.SeriesMasterId
              status: Topic.Status

          flowId: d9d7ccf7-7d61-f111-a826-6045bde03856

        - kind: SendActivity
          id: sendActivity_AAMZ3a
          displayName: C9_Confirm_SelectedMeeting
          activity: "Great — I've found your meeting: {Topic.MeetingTitle}"

        - kind: SetVariable
          id: setVariable_PageTitle01
          displayName: C9B_Set_PageTitle
          variable: Topic.PageTitle
          value: =Concatenate(Topic.MeetingTitle, " - ", Text(DateValue(Topic.DateContext), "d MMM yyyy"))

        - kind: InvokeFlowAction
          id: invokeFlowAction_bWHHeg
          displayName: C10_Call_FlowB_Create_Page
          input:
            binding:
              text: =Topic.IsRecurring
              text_1: =Topic.MeetingTitle
              text_2: =Topic.SeriesMasterId
              text_3: =Concatenate("<html><head><title>", Topic.PageTitle, "</title></head><body>", If(Topic.IsRecurring = "true", Concatenate("<p><strong>", Topic.MeetingTitle, "</strong></p>"), ""), If(Len(Topic.BodyPreview) > 0, Concatenate("<h2>Meeting Details</h2>", Topic.BodyPreview), ""), "<p>Meeting notes captured via Teams OneNote Meeting Capture agent.</p>", "</body></html>")
              text_4: =Topic.CalendarEventId
              text_5: =Topic.DateContext

          output:
            binding:
              outagentresponsesummary: Topic.OutAgentResponseSummary
              outbranchresult: Topic.OutBranchResult
              outcreatedpagelink: Topic.OutCreatedPageLink
              outcreatedpageselfurl: Topic.OutCreatedPageSelfUrl
              outexistingpageselfurl: Topic.OutExistingPageSelfUrl
              outfinaltargetsectionpagesurl: Topic.OutFinalTargetSectionPagesUrl
              outisrecurring: Topic.OutIsRecurring
              outmatchcount: Topic.OutMatchCount
              outmeetingtitle: Topic.OutMeetingTitle
              outonenoteresolverresult: Topic.OutOneNoteResolverResult
              outpageaction: Topic.OutPageAction
              outpagedecision: Topic.OutPageDecision
              outpagehtml: Topic.OutPageHtml
              outpageroute: Topic.OutPageRoute
              outresolverresult: Topic.OutResolverResult
              outseriesmasterid: Topic.OutSeriesMasterId
              outspitemcount: Topic.OutSPItemCount
              outstatus: Topic.OutStatus
              outtargetsectionpagesurl: Topic.OutTargetSectionPagesUrl
              outupdatehtmlfragment: Topic.OutUpdateHtmlFragment

          flowId: ed112c88-b94b-f111-bec6-002248a38052

        - kind: ConditionGroup
          id: conditionGroup_7KyK2P
          conditions:
            - id: conditionItem_zpJbO2
              condition: =Topic.OutStatus = "OK"
              displayName: C11_Check_OutStatus
              actions:
                - kind: SendActivity
                  id: sendActivity_VCuFOo
                  displayName: C12_Success
                  activity: "Great news! Your meeting notes for {Topic.MeetingTitle} have been saved to OneNote. Here's your page link: {Topic.OutCreatedPageLink}"

          elseActions:
            - kind: SendActivity
              id: sendActivity_nQYpmV
              displayName: C12_Error
              activity: I'm sorry, something went wrong saving your meeting notes. Please try again.

inputType: {}
outputType: {}
```

---

## Flow B — trigger schema

```json
{
  "type": "Request",
  "kind": "Skills",
  "inputs": {
    "schema": {
      "type": "object",
      "properties": {
        "text_1": { "title": "MeetingTitle", "type": "string", "x-ms-dynamically-added": true },
        "text_2": { "title": "SeriesMasterId", "type": "string", "x-ms-dynamically-added": true },
        "text_3": { "title": "PageHtml", "type": "string", "x-ms-dynamically-added": true },
        "text_4": { "title": "MeetingId", "type": "string", "x-ms-dynamically-added": true },
        "text": { "title": "IsRecurring", "type": "string", "x-ms-dynamically-added": true },
        "text_5": { "title": "OccurrenceDate", "type": "string", "x-ms-dynamically-added": true }
      },
      "required": ["text_1", "text_2", "text_3", "text_4", "text"]
    }
  }
}
```
Note: `text_5` (OccurrenceDate) intentionally not in `required` — optional field, matches documented design.

## Flow B — Get_items (SharePoint list query, top of flow)

```json
{
  "type": "OpenApiConnection",
  "inputs": {
    "parameters": {
      "dataset": "https://jsainsbury.sharepoint.com/sites/coplt",
      "table": "186b3c9f-e758-4e85-83d5-685946614a0a"
    },
    "host": {
      "apiId": "/providers/Microsoft.PowerApps/apis/shared_sharepointonline",
      "connection": "shared_sharepointonline",
      "operationId": "GetItems"
    }
  }
}
```
**⚠️ Confirmed 21 Aug: this GUID matches the list's actual current ID** (checked via List Settings URL). **⚠️ Confirmed 21 Aug: this action returned `"value": []` against a list containing at least one real row.** Root cause unresolved — see session note Part 4.

## Flow B — variable initializations (top of flow, in order)

`varFinalExistingPageSelfUrl` (string) → `varFinalMatchCount` (string) → `varOutStatus` (string) → `varOutputPageLink` (string) → `varOutputPageSelfUrl` (string) → `varTargetSectionPagesUrl` (string) → `varOneNoteResolverResult` (string) → `varPageAction` (string). All simple `InitializeVariable` actions, no default values, chained via `runAfter`.

## Flow B — Filter_Existing_Mapping container (FB-01, top-level branch)

```json
{
  "type": "If",
  "expression": { "and": [{ "equals": ["@equals(toLower(string(triggerBody()?['text'])), 'true')", true] }] },
  "actions": {
    "Compose_Input_SeriesMasterId": { "type": "Compose", "inputs": "@triggerBody()?['text_2']" },
    "Compose_Input_MeetingTitle": { "type": "Compose", "inputs": "@triggerBody()?['text_1']" },
    "Filter_Existing_Mapping": {
      "type": "Query",
      "inputs": {
        "from": "@body('Get_items')?['value']",
        "where": "@and(equals(item()?['SeriesMasterId'],triggerBody()?['text_2']),equals(item()?['OccurrenceDate'],triggerBody()?['text_5']))"
      }
    },
    "Compose_ExistingPageSelfUrl": {
      "type": "Compose",
      "inputs": "@if(greater(length(body('Filter_Existing_Mapping')), 0), first(body('Filter_Existing_Mapping'))?['PageSelfUrl'], '')"
    },
    "Compose_PageDecision": {
      "type": "Compose",
      "inputs": "@if(not(empty(outputs('Compose_ExistingPageSelfUrl'))), 'PAGE_EXISTS', 'PAGE_NOT_FOUND')"
    },
    "Compose_Match_Count": { "type": "Compose", "inputs": "@length(body('Filter_Existing_Mapping'))" },
    "varFinalExistingPageSelfUrl_1": { "type": "SetVariable", "inputs": { "name": "varFinalExistingPageSelfUrl", "value": "@outputs('Compose_ExistingPageSelfUrl')" } },
    "varFinalPageDecision_1": { "type": "SetVariable", "inputs": { "name": "varFinalPageDecision", "value": "@outputs('Compose_PageDecision')" } },
    "varFinalMatchCount_1": { "type": "SetVariable", "inputs": { "name": "varFinalMatchCount", "value": "@string(outputs('Compose_Match_Count'))" } }
  },
  "else": {
    "actions": {
      "__note__": "One-off (non-recurring) branch: FB-F01 title compose, Get/Filter/Create OneNote section, OF01-OF05c mapping filter (matches on MeetingId, not SeriesMasterId+OccurrenceDate). OF05c is the action that corrupted twice earlier this session (see driftcheck-and-corruption-incident.md) — confirmed restored and intact as of this capture."
    }
  }
}
```
(One-off branch actions omitted here for brevity — full detail preserved in the driftcheck session note; unchanged since that capture.)

## Flow B — Condition_Mapping_Exists (routes on varFinalMatchCount)

```json
{
  "type": "If",
  "expression": { "and": [{ "equals": ["@greater(int(if(empty(variables('varFinalMatchCount')), '0', variables('varFinalMatchCount'))), 0)", "@true"] }] },
  "actions": {
    "__true_branch__": "Compose_Branch_Result='EXISTS', sets varOneNoteResolverResult='ExistingMapping', Condition_Recurring_TargetSection sets varTargetSectionPagesUrl from Filter_Existing_Mapping's SectionPagesUrl",
    "__false_branch__": "Compose_Branch_Result_NoMatch='CREATE_REQUIRED', Condition_Should_Write_Mapping POSTs a new mapping row (Send_an_HTTP_request_to_SharePoint, includes OccurrenceDate), then resolves/creates the OneNote section (Get_Sections_Recurring / Filter_OneNote_Section_Recurring / Create_Section_Recurring), setting varOneNoteResolverResult to 'ExistingSection' or 'CreatedSection'"
  }
}
```
**This is the branch actually taken by today's test run** (Part 3): `Get_items` empty → `Filter_Existing_Mapping` empty → this condition's false branch → CREATE_REQUIRED path.

## Flow B — page creation/update container (contains FB-04)

```json
{
  "type": "If",
  "expression": { "and": [{ "equals": ["@equals(variables('varFinalPageDecision'), 'PAGE_NOT_FOUND')", true] }] },
  "actions": {
    "__true_branch_create__": "Compose_SafePageTitle (NOTE: title-only, no date — see session note Part 5), Create_OneNote_Page, Delay, Compose_PageSelfUrl_Created, Get_Pages_In_Section_Recurring_PostCreate, Filter_Pages_By_SelfUrl_Recurring, Compose_ConfirmedCreatedPageId, Set_PageTitle_Recurring, then OF09-Gate branches on IsRecurring to either update the SP mapping row's PageSelfUrl (recurring) or the one-off equivalent"
  },
  "else": {
    "actions": {
      "Set_varPageAction_ExistsNoCreate": "...",
      "Compose_UpdateHtmlFragment": "@concat('<hr><h2>Automated update</h2>...', triggerBody()?['text_3']) — confirmed intact, #2 fix",
      "Compose_ExistingPageId": "@last(split(variables('varOutputPageSelfUrl'), '/'))",
      "Condition_Is_Genuine_Existing_Page": {
        "true_branch": {
          "__contains_FB04__": true,
          "Filter_Existing_Section_By_Name": "...",
          "Apply_to_each_Existing_Section": {
            "Update_page_content_Existing_Branch": "appends Compose_UpdateHtmlFragment to page pageId=Compose_RealExistingPageId",
            "Get_Pages_In_Section_Existing_Branch": "...",
            "Filter_Pages_By_Title": {
              "where": "@contains(item()?['title'], formatDateTime(triggerBody()?['text_5'], 'd MMM yyyy'))",
              "__note__": "FB-04a — CONFIRMED correct via diff, 21 Aug. Not yet exercised by any live run."
            },
            "Compose_RealExistingPageId": {
              "inputs": "@if(greater(length(body('Filter_Pages_By_Title')), 0), first(body('Filter_Pages_By_Title'))?['id'], '')",
              "__note__": "FB-04b — CONFIRMED correct via diff, 21 Aug. Not yet exercised by any live run."
            }
          }
        },
        "else_branch": "one-off page creation (Create_Page_OneOff, Compose_SafePageTitle_OneOff, etc.) — this is the branch effectively exercised whenever Condition_Is_Genuine_Existing_Page evaluates false"
      }
    }
  }
}
```

## Flow B — response/output container

```json
{
  "type": "Response",
  "inputs": {
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
  }
}
```
**Note:** `outstatus` is hardcoded/derived from `varOutStatus`, which is set unconditionally to `"OK"` by `Set_varOutStatus` regardless of what actually happened (see `Compose_SP_Item_Count` / `Set_varOutStatus` below) — this is the existing, previously-documented "OutStatus never differentiates the six spec'd values" gap, unrelated to today's work but visible in this capture.

## Flow B — tail-end composes (AgentResponseSummary, SP item count, OutStatus)

```json
{
  "Compose_AgentResponseSummary": {
    "type": "Compose",
    "inputs": "@if(equals(variables('varPageAction'), 'Created'), concat('Created a new OneNote meeting page for \"', triggerBody()?['text_1'], '\".'), if(equals(variables('varPageAction'), 'UpdatedAppend'), concat('Updated the existing OneNote meeting page for \"', triggerBody()?['text_1'], '\" by appending a safe automated update block.'), if(equals(variables('varPageAction'), 'ExistsNoCreate'), concat('An existing OneNote meeting page was found for \"', triggerBody()?['text_1'], '\" and is ready for update.'), concat('Processed OneNote meeting page request for \"', triggerBody()?['text_1'], '\".'))))"
  },
  "Compose_SP_Item_Count": {
    "type": "Compose",
    "inputs": "@length(body('Get_items')?['value'])"
  },
  "Set_varOutStatus": {
    "type": "SetVariable",
    "inputs": { "name": "varOutStatus", "value": "OK" }
  }
}
```
**Note:** `Compose_SP_Item_Count` reads `length(body('Get_items')?['value'])` — on today's test run, this would have evaluated to `0`, directly reflecting the empty-list issue in `outspitemcount`. Worth checking this field on any future run as a quick health-check of the `Get_items` issue without needing to open the action itself.

---

*Captured 21 August 2026. Supersedes `flow-reference-2026-08-20-full-peek-code-capture.md`. Some sections abbreviated with `__note__`/`__branch__` markers where content is unchanged from the prior day's capture and already fully documented elsewhere — full verbatim JSON for those sections is preserved in the chat history and in `session-2026-08-21-driftcheck-and-corruption-incident.md` where relevant.*
