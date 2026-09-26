# Flow B v2 Recurring Child — Scope Map

**Locked:** 26 September 2026
**Variables:** target zero. **SetVariable:** none. All state via Compose + Filter Array.

Prefix scheme: `NZ` Normalize · `ML` MappingLookup · `EM` ExistingMapping · `NM` NewMapping · `PG` PageResolve · `WB` WriteBack · `ST` Status · `RP` Respond.

> **Build input still needed:** the full v1 recurring-branch Peek Code (runAfter + nesting) to map v1 actions precisely into these scopes. The known-good reference gives action *values* but not structure. The section/existing-page guard structure is confirmed; the mapping-vs-section branch nesting is not.

## Outer Scope: `Scope_FlowB_Recurring`

### `Scope_Normalize` (NZ)
**First: re-expose every trigger input as a named Compose** — `NZ00a Compose MeetingTitle` (`triggerBody()?['text']`), `NZ00b Compose SeriesMasterId` (`text_1`), `NZ00c Compose OccurrenceDate` (`text_2`), `NZ00d Compose PageHtml` (`text_3`), `NZ00e Compose EndTime` (`text_4`), `NZ00f Compose JoinUrl` (`text_5`). Everything downstream references these, never raw `text_N` (see trigger contract key-remap warning).
Then derive: `SafeSectionName` (prefix `Mtg -`, confirmed live 26 Sep), `SafePageTitle` (title + occurrence date), `UpdateHtmlFragment`, section display name.

### `Scope_MappingLookup` (ML)
- `ML01 Get items` — RecurringMeetingSectionMap, $top 500.
- `ML02 Filter Existing Mapping` (Filter Array / Query) — `SeriesMasterId == NZ00b AND OccurrenceDate == NZ00c`. Mapping rows are keyed per-occurrence (SeriesMasterId + OccurrenceDate), so a match means *this occurrence* was captured before.
- `ML03 Compose Match Count` — `@length(body('ML02_Filter_Existing_Mapping'))`.
- `ML04 Compose Mapping Exists` — `@greater(outputs('ML03_Compose_Match_Count'), 0)`.

### `Scope_ExistingMapping` (EM) — true branch of selector 1
Read the row via `first(body('ML02...'))?[...]`: SectionPagesUrl, ExistingPageSelfUrl, ExistingPageWebUrl, row ID. Resolver result = `ExistingMapping`. No SetVariable — single-row read is a Compose, not a loop.
- `EM01 Compose Is Stale Row` — `@empty(coalesce(first(body('ML02_Filter_Existing_Mapping'))?['SectionPagesUrl'], ''))`. Genuine stale test (matched row, blank section URL) — identical to UJ3b's `@empty(item()?['SectionPagesUrl'])`.
- **Branch selector 1a** (below): stale row short-circuits to Status (no page work); otherwise fall through to Page Resolve, using the row's SectionPagesUrl as the target section.

### `Scope_NewMapping` (NM) — false branch of selector 1
- Section resolution: `NM01 Get Sections` → `NM02 Filter Section By Name` (`name == SafeSectionName`) → `NM03 Compose Section Match Count`.
- **Branch selector 2** on count (below): create / use-existing / ambiguous.
- `NM04 Compose Target Section Pages Url` — `coalesce(created pagesUrl, first(existing)?['pagesUrl'])`.
- `NM05 Create Mapping Item Recurring` — PostItem incl. OccurrenceDate/JoinUrl/EndTime/SeriesMasterId/MeetingTitle/SectionPagesUrl/Status.

### `Scope_PageResolve` (PG)
**S1 duplicate-page guard re-implemented here** as a Filter-by-title check. **Every branch records `outpageaction`** — this is the fix for the v1 stale-fall-through bug.
- `PG01 Filter Pages By Title` (contains `OccurrenceDate` formatted `d MMM yyyy`) → `PG02 Compose Page Match Count`.
- **Branch selector 3** (below): append vs create.
  - Append → UpdatePageContent → `Compose PageAction` = `UpdatedAppend`.
  - Create → CreatePageInSection (into the target section — the row's SectionPagesUrl on the existing path, or NM04 on the new path) → `Compose PageAction` = `Created`.
- `PG0x Compose Page Self Url` / `Page Web Url` — coalesced across create/append result.

### `Scope_WriteBack` (WB)
Runs **whenever `PageAction == 'Created'`** — covers both a new occurrence (new mapping row) *and* a recovery-created page on an existing mapping whose stored page was gone. (Decision 4, refined — see flag below.)
- `WB00 Compose Target Row Id` — `coalesce(outputs('NM05...')?['body/ID'], first(body('ML02...'))?['ID'])` (new row, else existing row).
- `WB01 HTTP Update SP PageSelfUrl` (MERGE) — writes PageSelfUrl/PageWebUrl to that row.
- `WB02 Compose Mapping Write Succeeded` — `@equals(outputs('NM05...')?['statusCode'], 201)` (new-mapping path only) for PARTIAL_SUCCESS.

> **FLAG (confirm at build):** v1 does *not* write back a recovery-created page's URL on the existing-mapping path (the `Guard_Create_Page_Fallback` peek shows a create with no SharePoint update), leaving the row pointing at the old/dead page. Running WriteBack on any `Created` fixes that latent gap. Recommended — confirm you want the fix, or restrict WriteBack to the new-mapping path only for strict v1 parity.

### `Scope_Status` (ST)
Compose PageAction, ResolverResult, section match count, then `ST0x Compose OutStatus` — v1 nest rewritten to read Composes, not variables, with the stale clause redefined (precedence below). Also `ST0x Compose AgentResponseSummary`.
> **Build input:** v1's `Compose_AgentResponseSummary` expression is not in the known-good reference — pull it from the live v1 flow. It may be user-facing (the agent's summary line), so reproduce it faithfully rather than inventing one.

### `Scope_Respond` (RP)
Compose the 16 output fields; "Respond to a PowerApp or Flow." (Parent adds the 4 echoes.)

---

## Branch selectors

**Selector 1 — Mapping (ML):** `@greater(outputs('ML03_Compose_Match_Count'), 0)` — True ⇒ `Scope_ExistingMapping`; False ⇒ `Scope_NewMapping`.

**Selector 1a — Stale row (inside EM):** `@equals(outputs('EM01_Compose_Is_Stale_Row'), true)` — True ⇒ skip page work → Status (`STALE_MAPPING`); False ⇒ continue to `Scope_PageResolve`.

**Selector 2 — Section (inside NM), on `NM03 Compose Section Match Count`:**
- `0` ⇒ Create Section (resolver `CreatedSection`)
- `1` ⇒ Use existing (resolver `ExistingSection`)
- `>1` ⇒ `SETUP_SECTION_AMBIGUOUS`

**Selector 3 — Page (inside PG):** `@greater(outputs('PG02_Compose_Page_Match_Count'), 0)` — True ⇒ Append (`UpdatedAppend`); False ⇒ Create (`Created`).

---

## OutStatus precedence (recurring)

1. `EM01_Compose_Is_Stale_Row == true` → **`STALE_MAPPING`**
2. `PageAction ∈ {Created,Updated,UpdatedAppend}` AND `coalesce(MappingWriteSucceeded, 'true') == 'true'` → **`SUCCESS`**
3. `PageAction ∈ {...}` AND mapping write `== 'false'` → **`PARTIAL_SUCCESS`**
4. `empty(ResolverResult)` → **`RECURRING_SETUP_REQUIRED`**
5. `empty(TargetSectionPagesUrl)` → **`SETUP_SECTION_NOT_FOUND`**
6. `SectionMatchCount > 1` → **`SETUP_SECTION_AMBIGUOUS`**
7. else → **`ERROR`**

Mapping-write success defaults to `'true'` when absent (existing-mapping path has no Create_Mapping action), matching v1's `coalesce(..., 'true')`. This is why a successful existing-section append now returns `SUCCESS`, not `STALE_MAPPING`.

## Peek Code strategy
Outer `Scope_FlowB_Recurring` = full-flow pull. Per-phase pull on any inner Scope (ML / NM / PG especially). Push each inner Scope's Peek Code to `scope-peek-codes/` as it confirms green.
