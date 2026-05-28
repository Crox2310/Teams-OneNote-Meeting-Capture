# Agent Flow A v3.1 — Resolve Meeting Selection

## Physical flow name

```text
PA - Resolve Meeting Selection - v1 Clean Build
```

## Purpose

Flow A reads the user's Outlook calendar, finds candidate meetings, and returns clean selection data to the Copilot Studio topic.

## Scope

Flow A uses only:

```text
Office 365 Outlook — Get calendar view of events
```

Flow A does not use OneNote, SharePoint, Flow B, attendee extraction, or recurring setup in v1.

## Inputs

| Input | Type | Example | Value type |
|---|---|---|---|
| `InUserSearchText` | Text | `capture notes for my meeting` | Dynamic content from topic |
| `InDateContext` | Text | `today` | Plain text |
| `InMaxCandidates` | Text | `5` | Plain text |

## Outputs

All outputs are Text/String.

| Output | Notes |
|---|---|
| `Status` | `NO_MATCH`, `SINGLE_MATCH`, `MULTIPLE_MATCHES`, `ERROR` |
| `MatchCount` | Numeric count returned as string |
| `CandidateList` | Numbered readable list or empty string |
| `MeetingTitle` | Empty string if not single match |
| `CalendarEventId` | Empty string if not single match |
| `IsRecurring` | `"true"` or `"false"` as string |
| `SeriesMasterId` | May be empty in v1 |
| `Start` | Empty string if not single match |
| `End` | Empty string if not single match |
| `Location` | Empty string if not available |
| `AttendeesSummary` | Always empty string in v1 |
| `OutErrorMessage` | Empty unless error path triggered |

## Build action list

```text
FA00_Trigger_From_Agent
FA00A_Init_FA_VAR_Status
FA00B_Init_FA_VAR_MatchCount
FA00C_Init_FA_VAR_CandidateList
FA00D_Init_FA_VAR_MeetingTitle
FA00E_Init_FA_VAR_CalendarEventId
FA00F_Init_FA_VAR_IsRecurring
FA00G_Init_FA_VAR_SeriesMasterId
FA00H_Init_FA_VAR_Start
FA00I_Init_FA_VAR_End
FA00J_Init_FA_VAR_Location
FA00K_Init_FA_VAR_AttendeesSummary
FA00L_Init_FA_VAR_ErrorMessage
FA00M_Init_FA_VAR_CandidateIndex
FA01_Compose_StartWindowUtc
FA02_Compose_EndWindowUtc
FA03_O365_Get_Calendar_View_Events
FA03A_DEBUG_RawConnectorOutput
FA04_Compose_CalendarEventArray
FA05_Filter_Valid_Meetings
FA06_Compose_CandidateArray
FA07_Compose_MatchCountNumber
FA07A_Set_FA_VAR_MatchCount
FA08A_Condition_MatchCount_Equals_Zero
FA08A_TRUE_Set_Status_NoMatch
FA08A_TRUE_Set_MatchCount_Zero
FA08B_Condition_MatchCount_Equals_One
FA09_Compose_SingleMatchEvent
FA09A_Set_Status_SingleMatch
FA09B_Set_MeetingTitle
FA09C_Set_CalendarEventId
FA09D_Set_Start
FA09E_Set_End
FA09F_Set_Location
FA09G_Set_IsRecurring
FA09H_Set_SeriesMasterId
FA11_Set_Status_MultipleMatches
FA12_Reset_CandidateIndex
FA13_ApplyToEach_CandidateMeeting
FA13A_Increment_FA_VAR_CandidateIndex
FA13B_Append_To_FA_VAR_CandidateList
FA99_Respond_To_Agent
```

## Explicit exclusions from v1

Do not build:

```text
FA10_ApplyToEach_SingleMatchAttendees
FA10A_Append_Attendee_To_FA_VAR_AttendeesSummary
```

Attendee extraction is v2 enrichment.
