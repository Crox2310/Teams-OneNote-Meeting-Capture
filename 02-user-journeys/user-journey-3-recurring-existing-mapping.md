# User Journey 3 — Recurring Meeting, Existing Mapping Found

## Purpose

UJ3 handles a recurring meeting where a valid existing SharePoint mapping points to an accessible OneNote page.

## Key rules

- `SeriesMasterId` is treated as an opaque key.
- Topic applies Outlook Data Capture Profile V1 inclusion flags before PageHtml generation.
- Topic calls Flow B with `UpdateType = "NOTES"`.
- Flow B searches SharePoint mapping by `SeriesKey`.
- 0 rows → `RECURRING_SETUP_REQUIRED` → UJ4.
- 1 accessible row → append to existing OneNote page → `SUCCESS`, `PAGE_UPDATED_APPEND`.
- 1 stale row → `RECURRING_SETUP_REQUIRED` → UJ4.
- 2+ rows → `ERROR`.
