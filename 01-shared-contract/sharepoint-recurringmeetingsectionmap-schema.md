# SharePoint List Schema Reference — RecurringMeetingSectionMap

**Site:** jsainsbury.sharepoint.com/sites/coplt
**List:** RecurringMeetingSectionMap
**Purpose:** Maps a recurring meeting series (`SeriesMasterId`) to the OneNote section its notes are stored in, so UJ3 can find an existing mapping and UJ4 can create one.

## Confirmed internal column names

Captured directly from Flow B's create-item request body (`Send_an_HTTP_request_to_SharePoint`), 2026-07-20. These are the actual internal names the flow depends on — not display labels — since this project has twice been bitten by internal-vs-display name mismatches (`FA04`'s `DateContext`/`text_3` defect, AMEND-2026-07-18-001; the OneNote section-naming defect, AMEND-2026-07-19-004).

| Internal column name | Source in Flow B | Notes |
|---|---|---|
| `Title` | Literal `"Mapping"` | Standard SharePoint required column. Not used for lookup or identification — every row gets the same literal value. |
| `SeriesMasterId` | `outputs('Compose_Input_SeriesMasterId')` | The opaque recurring-series key (Decision Log vFinal, item 5). This is the field UJ3's mapping-exists check filters on. |
| `MeetingTitle` | `outputs('Compose_Input_MeetingTitle')` | Human-readable series title, for reference only. |
| `SectionPagesUrl` | `variables('varTargetSectionPagesUrl')` | The actual lookup target — the OneNote section's `pagesUrl`, used to create/append pages once a mapping is found. |
| `Status` | Literal `"Active"` | Set on every write; no other value has been observed. |

## Not present in the confirmed write body — needs verification

The following columns were referenced in earlier project documentation as part of this list's design, but do **not** appear in the create-item body captured above:

- `SectionDisplayName`
- `NotebookName`
- `CreatedAtUtc`
- `LastUsedAtUtc`

This could mean one of two things, and it isn't currently known which:

1. These columns exist in the SharePoint list but are simply never populated by Flow B — in which case they're dead weight in the schema, or a genuine gap if anything (e.g. staleness detection, per the UJ3 spec's "1 stale row" case) was meant to depend on them.
2. They're set by a different action elsewhere in Flow B not captured in this snippet — e.g. a separate update/patch step when an existing mapping is reused (which would explain `LastUsedAtUtc` specifically).

**Action needed:** check whether any other SharePoint write/update action exists in Flow B beyond the create-item step above (particularly on UJ3's "mapping exists, reuse" path), and confirm whether `CreatedAtUtc`/`LastUsedAtUtc` are being set there, are relying on SharePoint's built-in `Created`/`Modified` system columns instead, or are genuinely unused. This directly affects the still-open UJ3 gap around stale-row detection (see `2026-07-20-gap-analysis-original-brief-vs-current-build.md`, Section 5) — stale-row detection would most naturally key off `LastUsedAtUtc` if it exists and is actually maintained.

## Read-side usage (UJ3 mapping-exists check)

Not yet confirmed from a captured expression — the filter Flow B uses to query this list by `SeriesMasterId` (`Filter_Existing_Mapping` or equivalent) should be checked and added here for completeness, ideally alongside its exact OData filter string, since that's the other half of this list's read/write contract.
