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
| `outpageaction` | `Created` / `Updated` / `UpdatedAppend` / `''` — **recorded on every capture path** (empty only on the stale short-circuit) |
| `outupdatehtmlfragment` | Normalize Compose |
| `outagentresponsesummary` | Status Compose |
| `outstatus` | Status Compose (enum + precedence below) |

## `outstatus` reachable enum (recurring)
`SUCCESS` · `PARTIAL_SUCCESS` · `STALE_MAPPING` · `RECURRING_SETUP_REQUIRED` · `SETUP_SECTION_NOT_FOUND` · `SETUP_SECTION_AMBIGUOUS` · `ERROR`

**`STALE_MAPPING` redefined vs v1.** In v1 the existing-section guard never recorded a page action, so `Set_varOutStatus`'s stale clause (`empty(varPageAction) AND resolver ∈ {ExistingMapping,ExistingSection}`) caught *successful* existing-section captures and mislabelled them `STALE_MAPPING`. v2 records `outpageaction` on every path, so those return `SUCCESS`, and `STALE_MAPPING` is reserved for a real stale row:
```
@empty(coalesce(first(body('ML02_Filter_Existing_Mapping'))?['SectionPagesUrl'], ''))
```
i.e. a mapping row matched but its `SectionPagesUrl` is blank — identical to the UJ3b maintenance-flow definition. See `scope-map.md` for the full OutStatus precedence order.

**Topic-side note:** if the Meeting Capture Topic shows a different user-facing message for `STALE_MAPPING` vs `SUCCESS`, moving existing-section captures from STALE→SUCCESS changes what the user sees on those captures. Verify the Topic's OutStatus messaging at switchover — this is a Topic concern, not a C10 contract concern (keys are unchanged).
