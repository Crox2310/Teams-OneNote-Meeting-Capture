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
- `name`: `"Mtg - Antonio AL"`
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

### Full raw JSON — `foreachItems` (Apply_to_each_Existing_Section)

Captured in full for completeness / future reference:

```json
{
    "foreachItems": [
        {
            "id": "1-bd74643c-fc4d-410f-97ab-652c78bdce3a",
            "self": "https://www.onenote.com/api/v1.0/myOrganization/siteCollections/b5f8860c-4772-4e8b-b340-e80ba9d490fa/sites/d814850f-59bb-4182-92b7-e25d8c6a0487/notes/sections/1-bd74643c-fc4d-410f-97ab-652c78bdce3a",
            "createdTime": "2026-07-30T21:20:24Z",
            "name": "Mtg - Antonio AL",
            "createdBy": "David Croxson",
            "lastModifiedBy": "David Croxson",
            "lastModifiedTime": "2026-07-30T21:20:26Z",
            "isDefault": false,
            "links": {
                "oneNoteClientUrl": {
                    "href": "onenote:https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Meeting%20Notes/Mtg%20-%20Antonio%20AL.one"
                },
                "oneNoteWebUrl": {
                    "href": "https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Meeting%20Notes?wd=target%28Mtg%20-%20Antonio%20AL.one%2F%29"
                }
            },
            "pagesUrl": "https://www.onenote.com/api/v1.0/myOrganization/siteCollections/b5f8860c-4772-4e8b-b340-e80ba9d490fa/sites/d814850f-59bb-4182-92b7-e25d8c6a0487/notes/sections/1-bd74643c-fc4d-410f-97ab-652c78bdce3a/pages",
            "size": 6236,
            "parentNotebook@odata.context": "https://www.onenote.com/api/v1.0/$metadata#myOrganization/siteCollections('b5f8860c-4772-4e8b-b340-e80ba9d490fa')/sites('d814850f-59bb-4182-92b7-e25d8c6a0487')/notes/sections('1-bd74643c-fc4d-410f-97ab-652c78bdce3a')/parentNotebook(id,name,self)/$entity",
            "parentNotebook": {
                "id": "1-179a4384-615a-45a7-b96d-b3a3ba5d7b75",
                "name": "Meeting Notes",
                "self": "https://www.onenote.com/api/v1.0/myOrganization/siteCollections/b5f8860c-4772-4e8b-b340-e80ba9d490fa/sites/d814850f-59bb-4182-92b7-e25d8c6a0487/notes/notebooks/1-179a4384-615a-45a7-b96d-b3a3ba5d7b75"
            },
            "parentSectionGroup@odata.context": "https://www.onenote.com/api/v1.0/$metadata#myOrganization/siteCollections('b5f8860c-4772-4e8b-b340-e80ba9d490fa')/sites('d814850f-59bb-4182-92b7-e25d8c6a0487')/notes/sections('1-bd74643c-fc4d-410f-97ab-652c78bdce3a')/parentSectionGroup(id,name,self)/$entity"
        }
    ]
}
```

**Note on `parentSectionGroup`:** the `@odata.context` for `parentSectionGroup` is present but the `parentSectionGroup` value itself is absent from the payload — indicating the section sits directly under the notebook (`Meeting Notes`) rather than nested inside a section group. This rules out any section-group-nesting complication as a contributing factor to the `pageId` resolution defect.

## Second addendum — root cause pinpointed: variable name mismatch in `Compose ExistingPageId`

Opened `Compose ExistingPageId` directly (the action whose output feeds `pageId` into the failing call). Its full Code view definition:

```json
{
  "type": "Compose",
  "inputs": "@last(split(variables('varOutputPageSelfUrl'), '/'))",
  "runAfter": {
    "Compose_UpdateHtmlFragment": [
      "Succeeded"
    ]
  }
}
```

**This is the specific defect.** The expression reads `variables('varOutputPageSelfUrl')` — but the canvas trace immediately upstream shows the preceding Set action is named **`Set varOutputPageSelfUrlExisting`** (with the `Existing` suffix). These are two different variable names. `Compose ExistingPageId` is not reading the variable that the immediately-preceding step wrote to.

**Corroborating evidence from the same run:** `Compose ExistingPageId`'s own Run results (Inputs and Outputs panels, both `Show raw inputs` / `Show raw outputs`) are **both empty** — the action "succeeded" (0s, green check on canvas) but produced no meaningful value, consistent with `variables('varOutputPageSelfUrl')` evaluating to null/empty, `split(null, '/')` returning an empty result, and `last()` of that returning empty. This is exactly the empty `pageId` observed downstream in `Update_page_content_Existing_Branch`.

**This refines the working root-cause description.** Previous docs (29 July) described the defect at the level of "`varFinalExistingPageSelfUrl` never gets populated on the one-off path." Today's direct Code view inspection narrows it further: the specific expression at fault is `Compose ExistingPageId`'s reference to `varOutputPageSelfUrl` where the adjacent, correctly-populated variable is (or should be) `varOutputPageSelfUrlExisting`. Whether the underlying issue is a plain naming typo in the Compose expression, or whether `varOutputPageSelfUrl` and `varOutputPageSelfUrlExisting` are two genuinely distinct variables in the flow's variable list (in which case the real question becomes "why doesn't anything populate `varOutputPageSelfUrl` on this path"), is not yet confirmed — that requires a full listing of the flow's declared variables and every `Set` action that touches either name, across both branches.

**Not yet done / explicitly not concluded:** no edit has been made. The exact fix (repoint the expression at `varOutputPageSelfUrlExisting`, vs. some other correction) should not be applied until the full variable list and all `Set` actions referencing `varOutputPageSelfUrl*` are enumerated, per the standing pattern of confirming via Peek Code / live evidence before proposing fixes.

## Status

- **Root cause:** Pinpointed to a specific expression (30 July) — `Compose ExistingPageId` reads `variables('varOutputPageSelfUrl')`, not `varOutputPageSelfUrlExisting`, which the adjacent Set action populates. Empty Inputs/Outputs on `Compose ExistingPageId` in the same run corroborate this directly.
- **sectionId validity:** Confirmed genuine (30 July addendum) — not part of the defect. Full raw Section payload captured above for reference.
- **Fix:** Not yet implemented. Next action is to enumerate all `Set` actions and declared variables referencing `varOutputPageSelfUrl` / `varOutputPageSelfUrlExisting` (via "Go to operation" flat index, per established pattern) before editing the Compose expression, to confirm this is a straightforward naming fix and not a symptom of a wider variable-scoping issue.
- **Process note:** `amendment-log.md` backfill is still outstanding per the 20 July gap-analysis doc; this session's findings should be added to that backfill when it happens.
