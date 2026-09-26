# Flow B v2 Recurring Child — Output Contract

**Locked:** 26 September 2026
**Shape:** the 16 computed fields, identical across both children (parent owns the 4 echoes).

All fields `type: string`. Sourced from internal Composes — **no variables**.

| Key | Source (recurring) |
|---|---|
| `outspitemcount` | mapping-lookup count Compose |
| `outmatchcount` | `@string(length(body('ML02_Filter_Existing_Mapping')))` |
| `outbranchresult` | mirrors `outmatchcount` (Decision 7) |
| `outonenoteresolverresult` | `ExistingMapping` / `ExistingSection` / `CreatedSection` / `''` |
| `outtargetsectionpagesurl` | resolved section pages URL (Compose) |
| `outcreatedpagelink` | `oneNoteWebUrl` href from page create/append |
| `outcreatedpageselfurl` | page self-URL from page create |
| `outfinaltargetsectionpagesurl` | = `outtargetsectionpagesurl` |
| `outresolverresult` | = `outonenoteresolverresult` |
| `outexistingpageselfurl` | existing page self-URL Compose (or `''`) |
| `outpagedecision` | `PAGE_EXISTS` / `PAGE_NOT_FOUND` |
| `outpageroute` | `@string(equals(outputs('..PageDecision..'), 'PAGE_EXISTS'))` |
| `outpageaction` | `Created` / `Updated` / `UpdatedAppend` / `''` |
| `outupdatehtmlfragment` | Normalize Compose |
| `outagentresponsesummary` | Status Compose |
| `outstatus` | Status Compose (enum below) |

## `outstatus` reachable enum (recurring)
`SUCCESS` · `PARTIAL_SUCCESS` · `STALE_MAPPING` · `RECURRING_SETUP_REQUIRED` · `SETUP_SECTION_NOT_FOUND` · `SETUP_SECTION_AMBIGUOUS` · `ERROR`

Derived from Composes (not variables) mirroring the v1 `Set_varOutStatus` nest, simplified: the v1 `IsRecurring == 'true'` guard on `RECURRING_SETUP_REQUIRED` collapses to `empty(<resolver result>)` because this child is always recurring.
