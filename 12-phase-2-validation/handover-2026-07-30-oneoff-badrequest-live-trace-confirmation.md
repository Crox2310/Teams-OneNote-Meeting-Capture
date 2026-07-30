# Handover — 30 July 2026: Live-Trace Confirmation of One-Off Existing-Page BadRequest

## Purpose

This session captured a fresh live run of **PA - Resolve OneNote Meeting Section - v2 Clean Build** that failed with the same one-off existing-page `BadRequest` documented in:

- `handover-2026-07-29-oneoff-badrequest-investigation.md`
- `handover-2026-07-29-addendum-confirmed-root-cause.md`

The purpose of this doc is to log the corroborating evidence from today's run while fresh, before the fix is implemented. No fix was made in this session — investigation/confirmation only.

## Run details

- **Flow:** PA - Resolve OneNote Meeting Section - v2 Clean Build
- **Run:** 7/30/26, 10:20 PM (run ID `08584161604380972...`, truncated in UI)
- **Failing action:** `Update_page_content_Existing_Branch` (OneNote `UpdatePageContent` connector action)
- **Result:** `BadRequest`, statusCode `400`

## Evidence captured

**Action inputs (Run results → Show raw inputs):**
- `host.connectionReferenceName`: `shared_onenote`
- `host.operationId`: `UpdatePageContent`
- `parameters.notebookKey`: `Meeting Notes|$|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Meeting Notes`
- `parameters.sectionId`: `1-bd74643c-fc4d-410f-97ab-652c78bdce3a` (well-formed GUID, looks valid)
- **`parameters.pageId`: blank/empty**
- `parameters.updates`: single append operation with an HTML content fragment

**Action outputs (Run results → Show raw outputs):**
- `statusCode`: 400
- `body.error.code`: 400
- `body.error.message`: *"The section id given in the input is invalid. If a ..."* (truncated in UI, matches the previously logged error text)
- `body.error.path`: `choose[5]\when[1]`
- Standard Graph/connector response headers, tenant ID `e11fd634-26b5-47f4-8b8c-908e466e9bdf`, environment ID `76f9c3bd-16c5-e540-8bb4-7171f4745b45`

**Code view (static definition) confirms the expressions feeding those parameters:**
```json
"sectionId": "@items('Apply_to_each_Existing_Section')?['id']",
"pageId": "@outputs('Compose_ExistingPageId')"
```

Full static action definition (`Update_page_content_Existing_Branch`), captured via Code view, confirms the wrapping `Foreach` and `runAfter`:
```json
{
  "type": "Foreach",
  "foreach": "@body('Filter_Existing_Section_By_Name')",
  "actions": {
    "Update_page_content_Existing_Branch": {
      "type": "OpenApiConnection",
      "inputs": {
        "parameters": {
          "notebookKey": "Meeting Notes|$|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Meeting Notes",
          "sectionId": "@items('Apply_to_each_Existing_Section')?['id']",
          "pageId": "@outputs('Compose_ExistingPageId')",
          "updates": [
            {
              "target": "body",
              "action": "append",
              "position": "after",
              "content": "@outputs('Compose_UpdateHtmlFragment')"
            }
          ]
        },
        "host": {
          "apiId": "/providers/Microsoft.PowerApps/apis/shared_onenote",
          "connection": "shared_onenote",
          "operationId": "UpdatePageContent"
        }
      }
    }
  },
  "runAfter": {
    "Filter_Existing_Section_By_Name": [
      "Succeeded"
    ]
  }
}
```

**Upstream trace (canvas, same run):**
`Condition Should Create Page` → `False` branch → `Set varPageActionExistsNoCreate` → `Set varOutputPageSelfUrlExisting` → `Compose UpdateHtmlFragment` → `Compose ExistingPageId` → `Condition Is Genuine Existing Page` → `True` branch → `Get Sections Existing Branch` → `Filter Existing Section By Name` → `Apply to each Existing Section` → `Update page content Existing Branch` **(fails here)**.

## Addendum — sectionId confirmed genuine (from Apply_to_each_Existing_Section raw inputs)

Drilling into the parent `Apply to each Existing Section` loop's raw run inputs (`foreachItems`) shows the actual Section resource being iterated, item `[0]`:

- `id`: `1-bd74643c-fc4d-410f-97ab-652c78bdce3a` — **matches exactly** the `sectionId` passed into the failing `UpdatePageContent` call
- `name`: `"Mtg – Antonio AL"`
- `createdTime`: `2026-07-30T21:20:24Z`
- `createdBy` / `lastModifiedBy`: `David Croxson`
- `lastModifiedTime`: `2026-07-30T21:20:26Z`
- `isDefault`: `false`
- `links.oneNoteClientUrl` / `links.oneNoteWebUrl`: populated, pointing at the jsainsbury-my.sharepoint.com tenant
- `pagesUrl`: populated (Graph pages collection URL)
- `size`: `6236`
- `parentNotebook`: `id "1-179a4384-615a-45a7-b96d-b3a3ba5d7b75"`, `name "Meeting Notes"`

**This closes off the alternative hypothesis that `sectionId` itself was malformed or stale.** It is a real, currently-existing Section object returned live from Graph, correctly filtered by name, and correctly passed through to the failing call. This reinforces the existing conclusion: **`pageId` is the sole defective parameter** — Graph's "section id given in the input is invalid" message remains misleading.

**Worth flagging for the eventual fix investigation:** this section's `createdTime` (`21:20:24Z`) is only ~28 seconds before the flow run's start time (`10:20:52 PM` local / `21:20:52Z`, per the run's Start time property). The section was very recently created — plausibly by a prior step in this same capture cycle, or a near-simultaneous run. Not established as causal, but close enough in time that it's worth ruling in/out when tracing `varFinalExistingPageSelfUrl` — e.g. whether a race between section-creation propagation and the one-off page-resolution logic could be a contributing factor, separate from the already-confirmed "variable never populated" root cause.

## Status

- **Root cause:** Confirmed (29 July), corroborated again live (30 July) — same failure mode, same missing variable.
- **sectionId validity:** Confirmed genuine (30 July addendum) — not part of the defect.
- **Fix:** Not yet implemented.
- **Next step:** Trace `Set varOutputPageSelfUrlExisting` and confirm what populates (or fails to populate) `varFinalExistingPageSelfUrl` on the one-off path, per the recommended next steps in `handover-2026-07-29-addendum-confirmed-root-cause.md`. While there, also sanity-check whether section-creation timing/propagation could be a contributing factor per the note above.
- **Process note:** `amendment-log.md` backfill is still outstanding per the 20 July gap-analysis doc; this session's findings should be added to that backfill when it happens.
