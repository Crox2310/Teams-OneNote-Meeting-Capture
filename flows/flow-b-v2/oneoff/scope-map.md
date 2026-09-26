# Flow B v2 One-Off Child — Scope Map

**Locked:** 26 September 2026
**Variables:** target zero. **SetVariable:** none.

The fixed One-Off Meetings section removes the entire section-resolution phase — no Get Sections, no Create Section, no D2 branch. The old `Evt Mtg -` section-name Compose (v1 FB-F01) is dropped as dead.

Prefix scheme: `NZ` Normalize · `ML` MappingLookup · `EM` ExistingMapping · `NM` NewMapping · `PG` PageResolve · `WB` WriteBack · `ST` Status · `RP` Respond.

## Outer Scope: `Scope_FlowB_OneOff`

### `Scope_Normalize` (NZ)
**First: re-expose every trigger input as a named Compose** — `NZ00a Compose MeetingTitle` (`text`), `NZ00b Compose MeetingId` (`text_1`), `NZ00c Compose OccurrenceDate` (`text_2`), `NZ00d Compose PageHtml` (`text_3`), `NZ00e Compose EndTime` (`text_4`), `NZ00f Compose JoinUrl` (`text_5`). Downstream references these, never raw `text_N`.
Then derive: `SafePageTitle` (title + occurrence date), `UpdateHtmlFragment`, and `NZ0x Compose Target Section Pages Url` = the **hardcoded fixed One-Off Meetings section URL literal**. No `SafeSectionName`.

### `Scope_MappingLookup` (ML)
- `ML01 Get items` — RecurringMeetingSectionMap, $top 500.
- `ML02 Filter Existing Mapping OneOff` — `MeetingId == NZ00b AND OccurrenceDate == NZ00c`.
- `ML03 Compose Match Count` — `@length(...)`.
- `ML04 Compose Mapping Exists` — `@greater(outputs('ML03...'), 0)`.

### `Scope_ExistingMapping` (EM) — true branch
Read row via `first(...)?[...]`: PageSelfUrl, PageWebUrl, row ID. Resolver = `ExistingMapping`. (Section not read — fixed.)

### `Scope_NewMapping` (NM) — false branch
`NM01 Create Mapping Item OneOff` — PostItem incl. MeetingId/OccurrenceDate/JoinUrl/EndTime/MeetingTitle/SectionPagesUrl(fixed)/Status. **No section resolution.**

### `Scope_PageResolve` (PG)
S1 guard against the fixed section: `PG01 Filter Pages By Title` → `PG02 Compose Page Match Count` → **branch selector 2** append vs create (records `outpageaction` on both) → page self/web URL Composes.

### `Scope_WriteBack` (WB)
Runs **whenever `PageAction == 'Created'`** (new occurrence, or recovery-created page on an existing one-off row). `WB00 Compose Target Row Id` = `coalesce(new row ID, existing row ID)`; MERGE PageSelfUrl/PageWebUrl; `Compose Mapping Write Succeeded` for PARTIAL_SUCCESS. (Same recovery-write-back fix flagged in the recurring child — confirm at build.)

### `Scope_Status` (ST)
Compose PageAction, then `Compose OutStatus` — collapsed enum (SUCCESS / PARTIAL_SUCCESS / ERROR / [STALE_MAPPING kept, unreachable]). Also `Compose AgentResponseSummary` (pull v1's expression at build).

### `Scope_Respond` (RP)
Compose the 16 output fields (section fields = constants); "Respond to a PowerApp or Flow."

---

## Branch selectors

**Selector 1 — Mapping (ML):** `@greater(outputs('ML03_Compose_Match_Count'), 0)` — True ⇒ `Scope_ExistingMapping`; False ⇒ `Scope_NewMapping`.

**Selector 2 — Page (inside PG):** `@greater(outputs('PG02_Compose_Page_Match_Count'), 0)` — True ⇒ Append; False ⇒ Create.

*(No section selector — the whole simplification.)*

## Peek Code strategy
Outer `Scope_FlowB_OneOff` = full-flow pull; per-phase pull on ML / PG. Push each inner Scope's Peek Code to `scope-peek-codes/` as it confirms green.
