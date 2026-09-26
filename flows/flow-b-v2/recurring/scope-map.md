# Flow B v2 Recurring Child — Scope Map

**Locked:** 26 September 2026
**Variables:** target zero. **SetVariable:** none. All state via Compose + Filter Array.

Prefix scheme: `NZ` Normalize · `ML` MappingLookup · `EM` ExistingMapping · `NM` NewMapping · `PG` PageResolve · `WB` WriteBack · `ST` Status · `RP` Respond.

## Outer Scope: `Scope_FlowB_Recurring`

### `Scope_Normalize` (NZ)
Derive, pure Compose: `SafeSectionName` (prefix `Mtg -`, Decision 6), `SafePageTitle` (title + occurrence date), `UpdateHtmlFragment`, section display name.

### `Scope_MappingLookup` (ML)
- `ML01 Get items` — RecurringMeetingSectionMap, $top 500.
- `ML02 Filter Existing Mapping` (Filter Array / Query) — `SeriesMasterId == text_1 AND OccurrenceDate == text_2`.
- `ML03 Compose Match Count` — `@length(body('ML02_Filter_Existing_Mapping'))`.
- `ML04 Compose Mapping Exists` — `@greater(outputs('ML03_Compose_Match_Count'), 0)`.

### `Scope_ExistingMapping` (EM) — true branch of selector 1
Read the row via `first(body('ML02...'))?[...]`: SectionPagesUrl, ExistingPageSelfUrl, ExistingPageWebUrl. Resolver result = `ExistingMapping`. Feeds Page Resolve. No SetVariable — the single-row read is a Compose, not a loop.

### `Scope_NewMapping` (NM) — false branch of selector 1
- Section resolution: `NM01 Get Sections` → `NM02 Filter Section By Name` (`name == SafeSectionName`) → `NM03 Compose Section Match Count`.
- **Branch selector 2** on count (below): create / use-existing / ambiguous.
- `NM04 Compose Target Section Pages Url` — `coalesce(created pagesUrl, first(existing)?['pagesUrl'])`.
- `NM05 Create Mapping Item Recurring` — PostItem incl. OccurrenceDate/JoinUrl/EndTime/SeriesMasterId/MeetingTitle/SectionPagesUrl/Status.

### `Scope_PageResolve` (PG)
**S1 duplicate-page guard re-implemented here** as a Filter-by-title check.
- `PG01 Filter Pages By Title` (contains `OccurrenceDate` formatted `d MMM yyyy`) → `PG02 Compose Page Match Count`.
- **Branch selector 3** (below): append vs create.
- Dead-section fallback: if a mapped page id will not resolve, attempt create-in-section; if that also cannot proceed, the Status scope emits `STALE_MAPPING` (Decision 5/6 clarified during build).
- `PG0x Compose Page Self Url` / `Page Web Url` — coalesced across create/append result.

### `Scope_WriteBack` (WB)
Runs **only on new page / new mapping row** (Decision 4).
- `WB01 HTTP Update SP PageSelfUrl` (MERGE) — writes PageSelfUrl/PageWebUrl to the row.
- `WB02 Compose Mapping Write Succeeded` — `@equals(outputs('NM05...')?['statusCode'], 201)` for PARTIAL_SUCCESS.

### `Scope_Status` (ST)
Compose PageAction, ResolverResult, section match count, then `ST0x Compose OutStatus` — the v1 status nest rewritten to read Composes, not variables, simplified as noted in output-contract.md.

### `Scope_Respond` (RP)
Compose the 16 output fields; Respond. (Parent adds the 4 echoes.)

---

## Branch selectors

**Selector 1 — Mapping (ML):**
```
@greater(outputs('ML03_Compose_Match_Count'), 0)
```
True ⇒ `Scope_ExistingMapping`; False ⇒ `Scope_NewMapping`.

**Selector 2 — Section (inside NM), on `NM03 Compose Section Match Count`:**
- `0` ⇒ Create Section (resolver `CreatedSection`)
- `1` ⇒ Use existing (resolver `ExistingSection`)
- `>1` ⇒ `SETUP_SECTION_AMBIGUOUS`

**Selector 3 — Page (inside PG):**
```
@greater(outputs('PG02_Compose_Page_Match_Count'), 0)
```
True ⇒ Append (`Updated`/`UpdatedAppend`); False ⇒ Create (`Created`).

## Peek Code strategy
Outer `Scope_FlowB_Recurring` = full-flow pull. Per-phase pull on any inner Scope (ML / NM / PG especially). Push each inner Scope's Peek Code to `scope-peek-codes/` as it confirms green.
