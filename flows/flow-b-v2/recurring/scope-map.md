# Flow B v2 Recurring Child — Scope Map

**Locked:** 26 September 2026
**Variables:** target zero. **SetVariable:** none. All state via Compose + Filter Array.

Prefix scheme: `NZ` Normalize · `ML` MappingLookup · `EM` ExistingMapping · `NM` NewMapping · `PG` PageResolve · `WB` WriteBack · `ST` Status · `RP` Respond.

## Outer Scope: `Scope_FlowB_Recurring`

### `Scope_Normalize` (NZ)
Derive, pure Compose: `SafeSectionName` (prefix `Mtg -`, confirmed live 26 Sep), `SafePageTitle` (title + occurrence date), `UpdateHtmlFragment`, section display name.

### `Scope_MappingLookup` (ML)
- `ML01 Get items` — RecurringMeetingSectionMap, $top 500.
- `ML02 Filter Existing Mapping` (Filter Array / Query) — `SeriesMasterId == text_1 AND OccurrenceDate == text_2`.
- `ML03 Compose Match Count` — `@length(body('ML02_Filter_Existing_Mapping'))`.
- `ML04 Compose Mapping Exists` — `@greater(outputs('ML03_Compose_Match_Count'), 0)`.

### `Scope_ExistingMapping` (EM) — true branch of selector 1
Read the row via `first(body('ML02...'))?[...]`: SectionPagesUrl, ExistingPageSelfUrl, ExistingPageWebUrl. Resolver result = `ExistingMapping`. No SetVariable — the single-row read is a Compose, not a loop.
- `EM01 Compose Is Stale Row` — `@empty(coalesce(first(body('ML02_Filter_Existing_Mapping'))?['SectionPagesUrl'], ''))`. This is the **genuine** stale test (matched row, blank section URL) — identical to UJ3b's `@empty(item()?['SectionPagesUrl'])`.
- **Branch selector 1a** (below): stale row short-circuits to Status (no page work); otherwise fall through to Page Resolve.

### `Scope_NewMapping` (NM) — false branch of selector 1
- Section resolution: `NM01 Get Sections` → `NM02 Filter Section By Name` (`name == SafeSectionName`) → `NM03 Compose Section Match Count`.
- **Branch selector 2** on count (below): create / use-existing / ambiguous.
- `NM04 Compose Target Section Pages Url` — `coalesce(created pagesUrl, first(existing)?['pagesUrl'])`.
- `NM05 Create Mapping Item Recurring` — PostItem incl. OccurrenceDate/JoinUrl/EndTime/SeriesMasterId/MeetingTitle/SectionPagesUrl/Status.

### `Scope_PageResolve` (PG)
**S1 duplicate-page guard re-implemented here** as a Filter-by-title check. **Every branch records `outpageaction`** — this is the fix for the v1 stale-fall-through bug.
- `PG01 Filter Pages By Title` (contains `OccurrenceDate` formatted `d MMM yyyy`) → `PG02 Compose Page Match Count`.
- **Branch selector 3** (below): append vs create.
  - Append path → UpdatePageContent → `Compose PageAction` = `UpdatedAppend`.
  - Create path → CreatePageInSection → `Compose PageAction` = `Created`.
- `PG0x Compose Page Self Url` / `Page Web Url` — coalesced across create/append result.

### `Scope_WriteBack` (WB)
Runs **only on new page / new mapping row** (Decision 4).
- `WB01 HTTP Update SP PageSelfUrl` (MERGE) — writes PageSelfUrl/PageWebUrl to the row.
- `WB02 Compose Mapping Write Succeeded` — `@equals(outputs('NM05...')?['statusCode'], 201)` for PARTIAL_SUCCESS.

### `Scope_Status` (ST)
Compose PageAction, ResolverResult, section match count, then `ST0x Compose OutStatus` — the v1 nest rewritten to read Composes, not variables, with the stale clause redefined (see precedence below).

### `Scope_Respond` (RP)
Compose the 16 output fields; Respond. (Parent adds the 4 echoes.)

---

## Branch selectors

**Selector 1 — Mapping (ML):**
```
@greater(outputs('ML03_Compose_Match_Count'), 0)
```
True ⇒ `Scope_ExistingMapping`; False ⇒ `Scope_NewMapping`.

**Selector 1a — Stale row (inside EM):**
```
@equals(outputs('EM01_Compose_Is_Stale_Row'), true)
```
True ⇒ skip page work, go to Status (→ `STALE_MAPPING`); False ⇒ continue to `Scope_PageResolve`.

**Selector 2 — Section (inside NM), on `NM03 Compose Section Match Count`:**
- `0` ⇒ Create Section (resolver `CreatedSection`)
- `1` ⇒ Use existing (resolver `ExistingSection`)
- `>1` ⇒ `SETUP_SECTION_AMBIGUOUS`

**Selector 3 — Page (inside PG):**
```
@greater(outputs('PG02_Compose_Page_Match_Count'), 0)
```
True ⇒ Append (`UpdatedAppend`); False ⇒ Create (`Created`).

---

## OutStatus precedence (recurring)

Evaluated in this order (the `STALE_MAPPING` clause is redefined vs v1 — it is checked on the data condition, not on an empty page action):

1. `EM01_Compose_Is_Stale_Row == true` → **`STALE_MAPPING`**
2. `PageAction ∈ {Created,Updated,UpdatedAppend}` AND `coalesce(MappingWriteSucceeded, 'true') == 'true'` → **`SUCCESS`**
3. `PageAction ∈ {...}` AND mapping write `== 'false'` → **`PARTIAL_SUCCESS`**
4. `empty(ResolverResult)` → **`RECURRING_SETUP_REQUIRED`** (v1's `IsRecurring=='true'` guard collapses away — always recurring here)
5. `empty(TargetSectionPagesUrl)` → **`SETUP_SECTION_NOT_FOUND`**
6. `SectionMatchCount > 1` → **`SETUP_SECTION_AMBIGUOUS`**
7. else → **`ERROR`**

Note: mapping-write success defaults to `'true'` when absent (existing-mapping path has no Create_Mapping action), matching v1's `coalesce(..., 'true')`. This is why a successful existing-section append now returns `SUCCESS`, not `STALE_MAPPING`.

## Peek Code strategy
Outer `Scope_FlowB_Recurring` = full-flow pull. Per-phase pull on any inner Scope (ML / NM / PG especially). Push each inner Scope's Peek Code to `scope-peek-codes/` as it confirms green.
