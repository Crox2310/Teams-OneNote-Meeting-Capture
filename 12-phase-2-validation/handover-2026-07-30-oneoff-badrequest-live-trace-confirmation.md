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

**Worth flagging for the eventual fix investigation:** this section's `createdTime` (`21:20:24Z`) is only ~28 seconds before the flow run's start time (`10:20:52 PM` local / `21:20:52Z`, per the run's Start time property). The section was very recently created — plausibly by a prior step in this same capture cycle, or a near-simultaneous run. Not established as causal, but close enough in time that it's worth ruling in/out separately from the confirmed root cause below.

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

## Second addendum (superseded below) — initial hypothesis: variable name mismatch in `Compose ExistingPageId`

Opened `Compose ExistingPageId` directly. Its Code view definition:

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

At this point it looked like the expression referenced `varOutputPageSelfUrl` while the adjacent Set action was titled `Set varOutputPageSelfUrlExisting`, raising the hypothesis of a naming mismatch. **This hypothesis was ruled out** once the flow's declared variable list was pulled via "Go to operation" (see below) — there is no separate variable called `varOutputPageSelfUrlExisting`; that was only the action's display title. The single declared variable is `varOutputPageSelfUrl`, and the Compose expression correctly references it. The real defect is documented in the third addendum below.

## Third addendum — ROOT CAUSE CONFIRMED: missing `value` key on `Set varOutputPageSelfUrl Existing` (Pattern 6 recurrence)

Full variable-list sweep via "Go to operation" confirmed the flow's actual declared variables: `varTargetSectionPagesUrl`, `varOneNoteResolverResult`, `varPageAction`, `varOutputPageSelfUrl`, `varOutputPageLink`, `varOutStatus`, `varFinalExistingPageSelfUrl`, `varFinalPageDecision`, `varFinalMatchCount`. No `varOutputPageSelfUrlExisting` variable exists — confirming the second addendum's naming-mismatch theory was a dead end.

Opened `Set varOutputPageSelfUrl Existing` directly (the action immediately preceding `Compose ExistingPageId` on the one-off/existing-page path) and pulled its full Code view:

```json
{
  "type": "SetVariable",
  "inputs": {
    "name": "varOutputPageSelfUrl"
  },
  "runAfter": {
    "Set_varPageAction_ExistsNoCreate": [
      "Succeeded"
    ]
  }
}
```

**The `inputs` object has no `value` key at all.** The action sets the variable's name but never supplies an expression to assign to it.

**Confirmed at runtime** via this action's raw Inputs/Outputs for the 7/30 10:20 PM run:
```json
{
  "body": {
    "name": "varOutputPageSelfUrl",
    "type": "String",
    "value": ""
  }
}
```
Outputs: "No outputs" (SetVariable actions don't produce outputs; the effect is purely on the variable's state). `value` resolves to an empty string because there is nothing in the action definition to populate it with.

**This is the confirmed, complete causal chain:**
1. `Set varOutputPageSelfUrl Existing` — missing `value` key → sets `varOutputPageSelfUrl` to `""`
2. `Compose ExistingPageId` — `@last(split(variables('varOutputPageSelfUrl'), '/'))` → `split("", '/')` → `[""]` → `last()` → `""`
3. `Update_page_content_Existing_Branch` — `pageId: "@outputs('Compose_ExistingPageId')"` → empty `pageId`
4. Graph/OneNote connector — `BadRequest`, misleadingly reported against `sectionId`

**This is a recurrence of Pattern 6** from the living audit — "SetVariable actions missing `value` keys entirely" — previously identified and fixed twice on 27 July (AMEND-2026-07-27-001, AMEND-2026-07-27-002) elsewhere in this same flow. This specific action (`Set varOutputPageSelfUrl Existing`) was not part of that sweep and has now been found to carry the same defect.

**Not yet done:** no edit applied. The correct `value` expression for this action has not yet been determined — it should almost certainly reference the `self` URL of the matched existing section/page found earlier in the existing-page branch (candidates to check: output of `Get Sections Existing Branch` / `Filter Existing Section By Name`, or the declared-but-seemingly-unused `varFinalExistingPageSelfUrl` variable, which may be the intended source that was meant to be read here rather than reconstructed via this Compose/split pattern). This needs to be confirmed against the branch's upstream data shape before writing the fix.

## Status

- **Root cause:** **CONFIRMED** (30 July) — `Set varOutputPageSelfUrl Existing` is missing its `value` key entirely (Pattern 6), setting `varOutputPageSelfUrl` to `""`. This cascades through `Compose ExistingPageId` to produce an empty `pageId`, causing the `UpdatePageContent` `BadRequest`. Confirmed via Code view (static) and raw run Inputs (runtime) on the same failing run.
- **Ruled out:** sectionId validity (confirmed genuine — see addendum above); variable-name mismatch between `varOutputPageSelfUrl` and `varOutputPageSelfUrlExisting` (no such second variable exists — the "Existing" suffix is only an action title).
- **Fix:** Not yet implemented. Needs: determine the correct `value` expression for `Set varOutputPageSelfUrl Existing` (likely the matched section/page's `self` URL from upstream in the existing-page branch, or possibly `varFinalExistingPageSelfUrl` if that variable was intended to be the source and is itself unpopulated elsewhere — worth checking whether anything sets `varFinalExistingPageSelfUrl` at all, since no `Set varFinalExistingPageSelfUrl` action was seen in the full operation sweep).
- **Process note:** this should be logged as a new AMEND entry once fixed (following the AMEND-2026-07-27-001/002 numbering pattern), and `amendment-log.md` backfill (still outstanding per the 20 July gap-analysis doc) should include this alongside the earlier pattern-6 fixes.
