# Handover — 30 July 2026: Live-Trace Confirmation of One-Off Existing-Page BadRequest

## ⏭ START HERE NEXT SESSION

**Status: root cause confirmed, fix NOT started. This is a missing capability, not a quick patch.**

**The gap:** One-off (non-recurring) meetings have no mechanism anywhere in this flow to resolve "which existing OneNote page is this meeting's page" on recapture. Recurring meetings solve this via a SharePoint mapping table keyed on `SeriesMasterId` (which one-off meetings don't have). As a direct result, `varFinalExistingPageSelfUrl` stays `null` for one-off runs, `Set varOutputPageSelfUrl Existing` has nothing valid to assign, and the chain ends in a `BadRequest` on `Update_page_content_Existing_Branch` (Graph misleadingly blames `sectionId`; the real problem is an empty `pageId`).

**Next step is a design decision, not a code trace.** Two candidate directions are written up in the "Fourth addendum" section below — pick one (or propose a third) before touching any flow actions:
1. Extend the existing section-resolution loop (`Get Sections Existing Branch` → `Filter Existing Section By Name`) to also fetch pages within that section and match by title — no new mapping mechanism needed.
2. Build a new one-off-specific mapping mechanism (e.g. keyed on meeting subject + date, or Teams meeting ID) parallel to the recurring path's SharePoint table.

**Do not** fill in `Set varOutputPageSelfUrl Existing`'s blank value field with a guess or a placeholder before that decision is made — see "Fourth addendum" for why the obvious-looking fixes (naming mismatch, missing `value` key) turned out to be dead ends or symptoms rather than the real gap.

---

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

## Third addendum (refined further below) — missing `value` key on `Set varOutputPageSelfUrl Existing` (Pattern 6 recurrence)

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

At this point the working theory was that a straightforward `value` expression could be dropped into this action to fix it. **The fourth addendum below found this is not that simple.**

## Fourth addendum — REVISED FINDING: this is a missing capability, not a one-line fix. `varFinalExistingPageSelfUrl` is architecturally never populated for one-off meetings.

Traced backward from `Compose ExistingPageId` to find where a correct value for `varOutputPageSelfUrl` should originate.

**`Compose ExistingPageSelfUrl`** (Code view):
```json
"inputs": "@if(\n  greater(length(body('Filter_Existing_Mapping')), 0),\n  first(body('Filter_Existing_Mapping'))?['PageSelfUrl'],\n  ''\n)"
```
Pulls the `PageSelfUrl` column from the matched SharePoint mapping row, falling back to `''` if no match.

**`varFinalExistingPageSelfUrl`** — the declared variable's `InitializeVariable` action, positioned right after `Get items` near the top of the flow, before any recurring/one-off branching:
```json
"value": "@first(body('Filter_Existing_Mapping'))?['PageSelfUrl']"
```

**`varFinalExistingPageSelfUrl 1`** — a second, later `SetVariable` action:
```json
"value": "@outputs('Compose_ExistingPageSelfUrl')"
```

Both of these depend on `Filter_Existing_Mapping`, which (per the second addendum's earlier trace of `Set varTargetSectionPagesUrl ExistingMapping`) filters SharePoint rows by `SeriesMasterId` — a field that only exists for **recurring** meetings.

**Confirmed at runtime, on the actual failing run (7/30 10:20 PM):** `varFinalExistingPageSelfUrl`'s Run results (Inputs/Outputs of its `InitializeVariable` action) show:
```json
{
  "variables": [
    {
      "name": "varFinalExistingPageSelfUrl",
      "type": "String",
      "value": null
    }
  ]
}
```
It initializes to `null` and — on this one-off-meeting run — is never subsequently set to anything else.

**Confirmed via a full "Go to operation" search for "OneOff":** the complete list of one-off-specific actions is:
`Create Page OneOff`, `Get Sections OneOff`, `Create Section OneOff`, `Set varOutputPageLink Created OneOff`, `Filter OneNote Section OneOff`, `Condition Section Exists OneOff`, `Set varTargetSectionPagesUrl OneOff Exists`, `Set varOneNoteResolverResult Exists OneOff`, `Set varTargetSectionPagesUrl OneOff Created`, `Set varOneNoteResolverResult Created OneOff`, `Compose Section Match Count OneOff`, `FB-F01 — Compose Input MeetingTitle (one-off)`.

**None of these touch the SharePoint mapping table or resolve a specific existing page.** They are all about resolving or creating a **section** for a one-off meeting. There is no "Filter Existing Mapping OneOff" or equivalent page-level lookup for one-off meetings anywhere in the flow.

**Revised understanding of the defect:** This is not a single blank expression that can be filled in with an existing upstream value. The recurring-meeting path has a genuine data source for "the existing page for this meeting" — the SharePoint mapping table, keyed by `SeriesMasterId`, populated the first time that meeting series was captured. One-off meetings have no equivalent persistent key (`SeriesMasterId` doesn't exist for them), so there is **no mechanism anywhere in this flow that resolves a specific existing page for a one-off meeting on recapture.** The one-off path can find/recreate the right *section* (via the OneOff actions above), but nothing resolves the specific *page* within it. `Set varOutputPageSelfUrl Existing`'s blank `value` field is a symptom of this gap, not an isolated typo — there is currently nothing correct to put there for a one-off meeting.

**Open design question for the fix (not yet resolved):** how *should* the flow find "the existing page for this one-off meeting" on recapture, given there's no SeriesMasterId-based mapping to rely on? Candidate approaches to evaluate:
- Match by meeting title within the resolved section (the existing `Get Sections Existing Branch` → `Filter Existing Section By Name` → `Apply to each Existing Section` loop already narrows to a section — an equivalent "get pages in this section, filter/match by title or by SeriesMasterId absence" step could resolve the specific page, mirroring the recurring path's shape but without the mapping table).
- Some one-off-specific mapping mechanism (e.g. keyed on meeting subject + date, or Teams meeting ID) that doesn't currently exist and would need to be built new.

This needs a build/design decision, not just a Peek Code fix — recommend treating this as its own scoped piece of work rather than folding it into the pattern-6 amendment log entry.

## Status

- **Root cause:** **CONFIRMED, and revised in scope** (30 July). Immediate technical cause: `Set varOutputPageSelfUrl Existing` is missing its `value` key (Pattern 6), so `varOutputPageSelfUrl` = `""` → `Compose ExistingPageId` = `""` → empty `pageId` → `UpdatePageContent` `BadRequest` (misleadingly reported against `sectionId`). **Underlying cause:** there is no mechanism in this flow that resolves a specific existing OneNote page for a **one-off** meeting on recapture — the SharePoint-mapping-based resolution (`Filter_Existing_Mapping` → `varFinalExistingPageSelfUrl`) only works for recurring meetings (keyed on `SeriesMasterId`). Confirmed at runtime: `varFinalExistingPageSelfUrl` is `null` throughout the failing run, and no "OneOff" action anywhere in the flow performs an equivalent page-level lookup.
- **Ruled out:** sectionId validity (confirmed genuine); variable-name mismatch between `varOutputPageSelfUrl` and `varOutputPageSelfUrlExisting` (no such second variable exists).
- **Fix:** Not yet implemented, and now understood to require a design decision (see open design question above) rather than a single-expression patch. This should be scoped as its own piece of build work.
- **Process note:** the confirmed-but-narrower Pattern 6 defect (missing `value` key) is still worth its own AMEND log entry for traceability, but the fix itself should wait on resolving the underlying one-off-page-resolution gap — patching the blank field with a placeholder or the wrong source would likely just move the failure elsewhere. `amendment-log.md` backfill (still outstanding per the 20 July gap-analysis doc) should note both: the Pattern 6 finding, and this new open design item for one-off existing-page resolution.
