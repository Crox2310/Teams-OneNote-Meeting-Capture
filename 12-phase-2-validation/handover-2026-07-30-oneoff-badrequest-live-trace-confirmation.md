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

**Upstream trace (canvas, same run):**
`Condition Should Create Page` → `False` branch → `Set varPageActionExistsNoCreate` → `Set varOutputPageSelfUrlExisting` → `Compose UpdateHtmlFragment` → `Compose ExistingPageId` → `Condition Is Genuine Existing Page` → `True` branch → `Get Sections Existing Branch` → `Filter Existing Section By Name` → `Apply to each Existing Section` → `Update page content Existing Branch` **(fails here)**.

## Interpretation — consistent with 29 July confirmed root cause

This matches the confirmed root cause already logged on 29 July:

> `varFinalExistingPageSelfUrl` is never populated on the one-off path, causing `Compose ExistingPageId` to `split()` a null value, which cascades to `Condition Is Genuine Existing Page` failing / the eventual `UpdatePageContent` call receiving an empty `pageId`.

Today's evidence adds one clarifying detail not previously captured at this resolution: **the `sectionId` passed to `UpdatePageContent` is a well-formed, plausible-looking GUID — it is `pageId` that is empty.** Graph's error message ("section id given in the input is invalid") is misleading; the actual defect is the missing page ID, not the section ID. This is worth noting in the fix write-up so the eventual fix isn't mis-targeted at `sectionId` resolution logic, which is not defective.

## Status

- **Root cause:** Confirmed (29 July), corroborated again live (30 July) — same failure mode, same missing variable.
- **Fix:** Not yet implemented.
- **Next step:** Trace `Set varOutputPageSelfUrlExisting` and confirm what populates (or fails to populate) `varFinalExistingPageSelfUrl` on the one-off path, per the recommended next steps in `handover-2026-07-29-addendum-confirmed-root-cause.md`.
- **Process note:** `amendment-log.md` backfill is still outstanding per the 20 July gap-analysis doc; this session's findings should be added to that backfill when it happens.
