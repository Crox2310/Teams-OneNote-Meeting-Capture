# Scope_FlowB_Recurring — Confirmed Published Peek Code

**Flow:** PA - Meeting Capture - B - Recurring - v2
**Date confirmed:** 27 September 2026
**Status:** Published, 0 errors.
**Note:** Scope_PageAndRespond is an extra wrapper Scope containing Scope_PageResolve, Scope_WriteBack, Scope_Status, Scope_Respond. Functionally correct — Scope_FlowB_Recurring is still the outer health-check Scope.

## Trigger

```json
{
  "type": "Request",
  "kind": "PowerAppV2",
  "inputs": {
    "schema": {
      "properties": {
        "text":   { "title": "MeetingTitle",   "type": "string" },
        "text_1": { "title": "SeriesMasterId", "type": "string" },
        "text_2": { "title": "OccurrenceDate", "type": "string" },
        "text_3": { "title": "PageHtml",       "type": "string" },
        "text_4": { "title": "EndTime",        "type": "string" },
        "text_5": { "title": "JoinUrl",        "type": "string" }
      },
      "required": ["text", "text_1", "text_2", "text_3"]
    }
  }
}
```

## Outer Scope (Scope_FlowB_Recurring)

Full Peek Code as confirmed on 27 Sep 2026 — see raw JSON pasted to session for complete content. Key structure:

- Scope_Normalize (NZ00a–NZ04)
- Scope_MappingLookup (ML01–ML04)
- Condition_Mapping_Exists
  - True: Scope_ExistingMapping (EM01–EM06)
  - False: Scope_NewMapping (NM01–NM11)
- Scope_PageAndRespond
  - Scope_PageResolve (PG00–PG17)
  - Scope_WriteBack (WB00–WB03)
  - Scope_Status (ST01–ST03)
  - Scope_Respond (RP01–RP16 + RP_Respond)

## Key confirmed values

- NM05a CreateSectionInNotebook: `body/name` = `@outputs('NZ02_Compose_Safe_Section_Name')`, notebookKey = correct path
- NM02 filter: `@equals(item()?['name'], outputs('NZ02_Compose_Safe_Section_Name'))` (no extra quotes)
- PG11 operationId: `CreatePageInSection` (confirmed after multiple rebuilds)
- PG11 notebookKey: `Meeting Notes|$|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Meeting Notes` (no Master Archive Folder)
- WB02 headers: CustomHeader1/CustomHeader3/CustomHeader4 (matching one-off child)
- ST02 OutStatus: 7-clause if nest per scope-map.md precedence
- RP_Respond: 16 fields, kind PowerApp, statusCode 200

## Next step

Wire RT03a in the parent:
1. Open PA - Meeting Capture - B - Router - v2
2. False branch of RT02: add Run a Child Flow — name `RT03a Run Recurring Child` — select this flow
3. Map inputs per parent/trigger-contract.md
4. Set Scope_Relay runAfter Scope_Router to Succeeded AND Failed
5. Save, Flow Checker, publish, push updated scope-parent.md
