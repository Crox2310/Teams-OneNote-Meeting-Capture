# Flow B v2 One-Off Child — Output Contract

**Locked:** 26 September 2026
**Shape:** identical 16 computed fields to the recurring child (uniformity — Decision 2).

The fixed-section decision means the section-related fields are **constants**, not live resolution results. This is a deliberate behaviour change from v1's D2 branch — flagged, not accidental.

| Key | Source (one-off) |
|---|---|
| `outspitemcount` | mapping-lookup count Compose |
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
| `outpageroute` | `@string(equals(outputs('..PageDecision..'), 'PAGE_EXISTS'))` |
| `outpageaction` | `Created` / `Updated` / `UpdatedAppend` / `''` |
| `outupdatehtmlfragment` | Normalize Compose |
| `outagentresponsesummary` | Status Compose |
| `outstatus` | Status Compose (enum below) |

## `outstatus` reachable enum (one-off)
`SUCCESS` · `PARTIAL_SUCCESS` · `ERROR` — plus `STALE_MAPPING` kept in the enum for uniformity (Decision 5) though judged unreachable (fixed section always recovers via create). `SETUP_SECTION_*` and `RECURRING_SETUP_REQUIRED` cannot occur — no section resolution, not recurring.
