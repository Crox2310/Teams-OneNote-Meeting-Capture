# Session note addendum — 22 August 2026 (late afternoon)

**Appended to:** full day session notes.

---

## Part 8 — FA16 live verification: confirmed

Triggered a capture via Teams, selected meeting number `2` from the candidate list (P7 W1 Week 25). Agent correctly identified the meeting, capture succeeded, clickable OneNote link returned. FA16's defensive guard did not interfere with normal numeric selection. ✅

Bonus confirmation: the OneNote link returned was a proper clickable `jsainsbury-my.sharepoint.com` web URL, confirming the link-format bug fix is also working correctly on the one-off path.

**FA16 live verification: closed.** ✅

---

## Part 9 — BadGateway fix: native Create item connector

Replaced `Send an HTTP request to SharePoint` (raw REST POST) on both the recurring and one-off mapping-write paths with the native SharePoint **Create item** connector action. The native connector handles throttling and retries more gracefully than raw HTTP, which was the root cause of the intermittent 502 BadGateway errors.

**Model/effort used:** Sonnet 4.6, Medium effort.

### Changes made (six in total, all Peek Code verified)

**Recurring path:**

1. **Deleted** `Send_an_HTTP_request_to_SharePoint` (raw HTTP POST)
2. **Added** `Create_Mapping_Item_Recurring` (native SharePoint Create item, `PostItem` operation) with fields:
   - `Title` = `Mapping`
   - `SeriesMasterId` = `outputs('Compose_Input_SeriesMasterId')`
   - `MeetingTitle` = `outputs('Compose_Input_MeetingTitle')`
   - `SectionPagesUrl` = `variables('varTargetSectionPagesUrl')`
   - `Status/Value` = `Active`
   - `OccurrenceDate` = `triggerBody()?['text_5']`
3. **Updated** `Compose_MappingWriteSucceeded` — new action reference and status code `200` (native connector returns 200, not 201):
```
@if(equals(outputs('Create_Mapping_Item_Recurring')?['statusCode'], 200), 'true', 'false')
```
   Run after: all four statuses (SUCCEEDED/FAILED/TIMEDOUT/SKIPPED) ✅
4. **Updated** `HTTP_Update_SP_PageSelfUrl` URI — replaced `body('Send_an_HTTP_request_to_SharePoint')?['ID']` with `body('Create_Mapping_Item_Recurring')?['ID']`

**One-off path:**

5. **Deleted** `OF09a_—_Send_an_HTTP_request_to_SharePoint_(OneOff)` (raw HTTP POST)
6. **Added** `Create_Mapping_Item_OneOff` (native SharePoint Create item) with fields:
   - `Title` = `Mapping`
   - `MeetingId` = `triggerBody()?['text_4']`
   - `MeetingTitle` = `outputs('FB-F01_—_Compose_Input_MeetingTitle_(one-off)')`
   - `SectionPagesUrl` = `variables('varTargetSectionPagesUrl')`
   - `Status/Value` = `Active`
   - Note: no `SeriesMasterId` or `OccurrenceDate` on one-off path — correct by design
7. **Updated** `Compose_MappingWriteSucceeded_OneOff` — new action reference and status code `200`:
```
@if(equals(outputs('Create_Mapping_Item_OneOff')?['statusCode'], 200), 'true', 'false')
```
   Run after: all four statuses ✅
8. **Updated** `OF09b_—_HTTP_Update_SP_PageSelfUrl_(OneOff)` URI — replaced `body('OF09a_—_Send_an_HTTP_request_to_SharePoint_(OneOff)')?['ID']` with `body('Create_Mapping_Item_OneOff')?['ID']`

Flow checker 0 errors. Published. ✅

**Verification test pending** — to be run next to confirm mapping row writes cleanly with the native connector.

---
