# Flow B v2 One-Off Child — Output Contract

**Locked:** 26 September 2026
**Shape:** identical 16 computed fields to the recurring child (uniformity — Decision 2).

The fixed-section decision means the section-related fields are **constants**, not live resolution results. Deliberate behaviour change from v1's D2 branch — flagged, not accidental. Where a field has a fixed v1 expression *form*, reproduce that form (see recurring output-contract for `outpageroute` / `outspitemcount`).

| Key | Source (one-off) |
|---|---|
| `outspitemcount` | `@{int(coalesce(outputs('<SP item count compose>'), 0))}` (v1 form) |
| `outmatchcount` | `@string(length(body('ML02_Filter_Existing_Mapping_OneOff')))` |
| `outbranchresult` | mirrors `outmatchcount` (Decision 7) |
| `outonenoteresolverresult` | **constant** (`''` or fixed label) — no live section resolution |
| `outtargetsectionpagesurl` | **fixed One-Off Meetings section URL (constant)** |
| `outcreatedpagelink` | `oneNoteWebUrl` href from page create/append |
| `outcreatedpageselfurl` | page self-URL from page create |
| `outfinaltargetsectionpagesurl` | = `outtargetsectionpagesurl` (constant) |
| `outresolverresult` | = `outonenoteresolverresult` (constant) |
| `outexistingpageselfurl` | existing page self-URL Compose (or `''`) |
| `outpagedecision` | `PAGE_EXISTS` / `PAGE_NOT_FOUND` |
| `outpageroute` | `@{equals(outputs('<PageDecision compose>'), 'PAGE_EXISTS')}` (v1 form) |
| `outpageaction` | `Created` / `Updated` / `UpdatedAppend` / `''` — recorded on every capture path |
| `outupdatehtmlfragment` | Normalize Compose |
| `outagentresponsesummary` | Status Compose — reproduce v1's `Compose_AgentResponseSummary` |
| `outstatus` | Status Compose (enum below) |

## `outstatus` reachable enum (one-off)
`SUCCESS` · `PARTIAL_SUCCESS` · `ERROR` — plus `STALE_MAPPING` kept for uniformity (Decision 5) though **unreachable**: the stale test is `@empty(coalesce(first(...)?['SectionPagesUrl'], ''))`, but the one-off target section is the fixed constant URL and is never blank, so the condition cannot trip. `SETUP_SECTION_*` and `RECURRING_SETUP_REQUIRED` cannot occur — no section resolution, not recurring.
