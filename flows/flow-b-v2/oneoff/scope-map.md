# Flow B v2 One-Off Child — Scope Map

**Locked:** 26 September 2026
**Variables:** target zero. **SetVariable:** none.

The fixed One-Off Meetings section removes the entire section-resolution phase — no Get Sections, no Create Section, no D2 branch. The old `Evt Mtg -` section-name Compose (v1 FB-F01) is dropped as dead.

Prefix scheme: `NZ` Normalize · `ML` MappingLookup · `EM` ExistingMapping · `NM` NewMapping · `PG` PageResolve · `WB` WriteBack · `ST` Status · `RP` Respond.

## Outer Scope: `Scope_FlowB_OneOff`

### `Scope_Normalize` (NZ)
Pure Compose: `SafePageTitle` (title + occurrence date), `UpdateHtmlFragment`, and `NZ0x Compose Target Section Pages Url` = the **hardcoded fixed One-Off Meetings section URL literal**. No `SafeSectionName`.

### `Scope_MappingLookup` (ML)
- `ML01 Get items` — RecurringMeetingSectionMap, $top 500.
- `ML02 Filter Existing Mapping OneOff` — `MeetingId == text_1 AND OccurrenceDate == text_2`.
- `ML03 Compose Match Count` — `@length(...)`.
- `ML04 Compose Mapping Exists` — `@greater(outputs('ML03...'), 0)`.

### `Scope_ExistingMapping` (EM) — true branch
Read row via `first(...)?[...]`: PageSelfUrl, PageWebUrl. Resolver = `ExistingMapping`. (Section not read — fixed.)

### `Scope_NewMapping` (NM) — false branch
`NM01 Create Mapping Item OneOff` — PostItem incl. MeetingId/OccurrenceDate/JoinUrl/EndTime/MeetingTitle/SectionPagesUrl(fixed)/Status. **No section resolution.**

### `Scope_PageResolve` (PG)
S1 guard against the fixed section: `PG01 Filter Pages By Title` → `PG02 Compose Page Match Count` → **branch selector 2** append vs create → page self/web URL Composes.

### `Scope_WriteBack` (WB)
Runs only on new page / new row (Decision 4). MERGE PageSelfUrl/PageWebUrl to the one-off row; `Compose Mapping Write Succeeded` for PARTIAL_SUCCESS.

### `Scope_Status` (ST)
Compose PageAction, then `Compose OutStatus` — collapsed enum (SUCCESS / PARTIAL_SUCCESS / ERROR / [STALE_MAPPING kept, unreachable]).

### `Scope_Respond` (RP)
Compose the 16 output fields (section fields = constants); Respond.

---

## Branch selectors

**Selector 1 — Mapping (ML):**
```
@greater(outputs('ML03_Compose_Match_Count'), 0)
```
True ⇒ `Scope_ExistingMapping`; False ⇒ `Scope_NewMapping`.

**Selector 2 — Page (inside PG):**
```
@greater(outputs('PG02_Compose_Page_Match_Count'), 0)
```
True ⇒ Append; False ⇒ Create.

*(No section selector — the whole simplification.)*

## Peek Code strategy
Outer `Scope_FlowB_OneOff` = full-flow pull; per-phase pull on ML / PG. Push each inner Scope's Peek Code to `scope-peek-codes/` as it confirms green.
