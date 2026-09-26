# Scope_MultiMatch — Correction to Build Instructions

**Date:** 26 September 2026
**Reason:** indexOf() confirmed not supported for arrays. Build instructions in build-instructions.md still show the old indexOf() approach in MM04c. Use THIS file for Scope_MultiMatch instead.

---

## Scope_MultiMatch — correct build steps

**S05 — Add inner Scope.** Inside `Scope_FlowA`, add a Scope. Name it `Scope_MultiMatch`.

**MM01 — Condition Is Multi Match.**
- Type: Condition
- Name: `MM01 Condition Is Multi Match`
- Left (Expression): `outputs('CR01_Compose_Match_Count')`
- Operator: greater than
- Right (Expression): `1`

**Inside MM01 True branch:**

**MM02 — Get Mapping Rows.**
- Type: SharePoint — Get items
- Name: `MM02 Get Mapping Rows`
- Site Address (Expression): `'https://jsainsbury.sharepoint.com/sites/coplt'`
- List Name (Expression): `'186b3c9f-e758-4e85-83d5-685946614a0a'`
- Filter Query (Expression): `OccurrenceDate eq '@{formatDateTime(outputs('CA01_Compose_DateContext'), 'yyyy-MM-dd')}'`
- Top Count (Expression): `50`

**MM03 — Initialise Candidate List Text.**
- Type: Initialize Variable
- Name: `MM03 Initialise Candidate List Text`
- Variable name: `varCandidateListText`
- Type: String
- Value: leave blank

**MM03b — Initialise Candidate Index.**
- Type: Initialize Variable
- Name: `MM03b Initialise Candidate Index`
- Variable name: `varCandidateIndex`
- Type: Integer
- Value: `1`

⚠️ Both MM03 and MM03b are the only two variables in v2. Both must sit here immediately before the loop, not at the top of the flow.

**MM04 — Build Candidate List Loop.**
- Type: For each
- Name: `MM04 Build Candidate List Loop`
- Select output (Expression): `outputs('CA07_Compose_Sorted_Candidates')`

**Inside MM04 loop — add in order:**

**MM04a — Filter Mapping Match.**
- Type: Filter Array
- Name: `MM04a Filter Mapping Match`
- From (Expression): `body('MM02_Get_Mapping_Rows')?['value']`
- Filter (advanced mode Expression): `@or(equals(item()?['SeriesMasterId'], coalesce(items('MM04_Build_Candidate_List_Loop')?['seriesMasterId'], '')), equals(item()?['MeetingId'], coalesce(items('MM04_Build_Candidate_List_Loop')?['id'], '')))`

**MM04b — Compose Status Label.**
- Type: Compose
- Name: `MM04b Compose Status Label`
- Value (Expression): `if(equals(coalesce(first(body('MM04a_Filter_Mapping_Match'))?['RecapCaptured'], false), true), ' - **Mtg Notes**', if(greater(length(body('MM04a_Filter_Mapping_Match')), 0), ' - **Captured**', ''))`

**MM04c — Append To Candidate List.**
- Type: Append to string variable
- Name: `MM04c Append To Candidate List`
- Variable name: `varCandidateListText`
- Value (Expression): `concat(string(variables('varCandidateIndex')), '. ', coalesce(items('MM04_Build_Candidate_List_Loop')?['subject'], 'Untitled meeting'), outputs('MM04b_Compose_Status_Label'), decodeUriComponent('%0D%0A'))`

**MM04d — Increment Candidate Index.**
- Type: Increment Variable
- Name: `MM04d Increment Candidate Index`
- Variable name: `varCandidateIndex`
- Value: `1`

**After MM04 loop (still inside MM01 True branch):**

| Ref | Name | Expression |
|---|---|---|
| MM05 | Compose Multi Status | `'MULTIPLE_MATCHES'` |
| MM06 | Compose Multi Match Count | `string(outputs('CR01_Compose_Match_Count'))` |
| MM07 | Compose Multi Candidate List | `concat('Meetings for ', outputs('CR02_Compose_Display_Date'), decodeUriComponent('%0D%0A'), variables('varCandidateListText'))` |
| MM08 | Compose Multi Title | `string('')` |
| MM09 | Compose Multi EventId | `string('')` |
| MM10 | Compose Multi IsRecurring | `string('')` |
| MM11 | Compose Multi SeriesMasterId | `string('')` |
| MM12 | Compose Multi JoinUrl | `string('')` |
| MM13 | Compose Multi EndTime | `string('')` |

Leave MM01 False branch empty.

**Scope_MultiMatch — corruption recovery (mandatory before next Scope):**
1. Peek Code `Scope_MultiMatch` → push to `scope-peek-codes/scope-multi-match.md`
2. Update `known-good-values.md` with MM01–MM13
3. Save draft → close/reopen → wait 20s → Flow Checker 0 errors → Publish
