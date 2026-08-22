# Session note addendum — 22 August 2026 (BadGateway verification and status code fix)

---

## BadGateway fix verification: completed, with one correction needed

### First verification run

Capture of "New Repeat Meeting" (new recurring series, 22 Aug 2026 occurrence). Results:
- OneNote page created with correct dated title `New Repeat Meeting - 22 Aug 2026` ✅
- SharePoint mapping row written with `OccurrenceDate: 2026-08-22`, `SeriesMasterId` and `MeetingTitle` populated ✅ — **BadGateway confirmed resolved** by the native connector switch
- `Create_Mapping_Item_Recurring` completed in 0.3s with green tick, no retries ✅
- **BUT:** Agent returned error message `"I'm sorry, something went wrong"` ❌
- `Compose_MappingWriteSucceeded` output `"false"` despite the Create item action succeeding
- `Set_varOutStatus` evaluated to `PARTIAL_SUCCESS` instead of `SUCCESS`

### Root cause: wrong status code in Compose_MappingWriteSucceeded

The expression checked for status code `200` but the native SharePoint Create item connector (`PostItem`) returns `201 Created` on success (standard HTTP semantics for a newly created resource). The old raw HTTP POST also returned `201`, but this was incorrectly changed to `200` when building the Compose expression. Result: the Compose always output `'false'`, causing `OutStatus` to evaluate to `PARTIAL_SUCCESS` even on a fully successful run.

### Fix applied

Both Compose actions updated from `200` to `201`:

**`Compose_MappingWriteSucceeded`:**
```
@if(equals(outputs('Create_Mapping_Item_Recurring')?['statusCode'], 201), 'true', 'false')
```

**`Compose_MappingWriteSucceeded_OneOff`:**
```
@if(equals(outputs('Create_Mapping_Item_OneOff')?['statusCode'], 201), 'true', 'false')
```

Both confirmed via Peek Code diff. All four `runAfter` statuses intact. Flow checker 0 errors. Published.

### Second verification run

Same series, same occurrence. Results:
- Agent responded with success message and clickable OneNote link ✅
- Page `New Repeat Meeting - 22 Aug 2026` visible in OneNote ✅
- `OutStatus = "SUCCESS"` confirmed ✅

**BadGateway fix: fully verified and closed.** ✅

---
