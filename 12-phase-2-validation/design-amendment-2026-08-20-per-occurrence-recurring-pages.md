# Design amendment — per-occurrence pages for recurring meetings (20 August 2026)

## Status
**Proposed — not yet built.** This document captures the confirmed requirement, the evidence behind it, and the recommended approach. No flow changes have been made yet.

## Origin
Raised by David during live use of the agent on 20 August 2026 (three field observations in one session — this doc covers observation 1 only; see companion notes for observations 2 and 3).

## The gap

**Current behaviour (confirmed by evidence, not assumption):**

The recurring branch of Flow B (`PA - Resolve OneNote Meeting Section - v2 Clean Build`) tracks **one OneNote page per recurring series**, keyed solely on `SeriesMasterId`:

- `Filter_Existing_Mapping` queries the `RecurringMeetingSectionMap` SharePoint list on `item()?['SeriesMasterId'] == triggerBody()?['text_2']` only.
- `SeriesMasterId` is constant across every occurrence of a series (confirmed: identical value `AAMkAGY0OGU4Mzk5LWQ4NTYtNDU4MS1hY2YyLTQxOWYwZjhiMWM1ZQBGAAAAAADWkXK1vW2mQ4SwNGpyD7SzBwB8mPnOPkRmT5-MxNoNopoPAAAAAAENAAB8mPnOPkRmT5-MxNoNopoPAAXZ7xgbAAA= ` across two live captures of the "121 Simon / David" series, one week apart).
- Result: a second capture of a recurring meeting is treated as an **update** to the single existing page (append via `Compose_UpdateHtmlFragment` / `Update_page_content_Existing_Branch`), never as a new page.

**Live-tested and confirmed 20 August 2026:** capturing "121 Simon / David" twice (18:00 and 18:06) produced no error and no second page — the second capture appended a new content block (Teams join details) below the original content on the same single page. SharePoint mapping row and OneNote section both show exactly one page after two captures.

**Desired behaviour (David, 20 Aug 2026):**
> "I think each [occurrence] should get their own page, if the date is the same it should be an append update as it might be that the meeting organiser has updated meeting content/details."

So:
- **Different occurrence date** → create a **new page**, titled with meeting name + date (e.g. "121 Simon / David — 2 Sep 2026").
- **Same occurrence date** captured again → **append/protect update** into that date's existing page (organiser may have changed meeting content).

## Evidence: the data needed already exists in the trigger payload

No changes to Flow A or the Topic are required. Two fields already arrive at Flow B that vary correctly per occurrence:

| Field | Run 1 (19 Aug meeting, captured 20 Aug 18:00) | Run 2 (2 Sep meeting, captured 20 Aug 18:06) | Varies per occurrence? |
|---|---|---|---|
| `text_2` (`SeriesMasterId`) | `...QBGAAAAAADWkXK1...` | `...QBGAAAAAADWkXK1...` (identical) | No — constant per series |
| `text_4` (`MeetingId`) | `...QFRAAgI3v2Ez-qAAEYAAAAA1pFytb1...` | `...QFRAAgI3wiFIcMAAEYAAAAA1pFytb1...` | **Yes** — unique per occurrence (shared prefix/suffix, differing middle segment) |
| `text_3` (`PageHtml`) `<title>` tag | `<title>19 Aug 2026</title>` | `<title>2 Sep 2026</title>` | **Yes** — human-readable occurrence date, confirmed correct against actual meeting dates |

Both `MeetingId` and the embedded date are currently **received but unused** by the recurring branch (the one-off branch already uses `MeetingId` via `OF01_Filter_Existing_Mapping_OneOff`; the recurring branch uses neither).

## Recommendation

Use the **date extracted from `text_3`'s `<title>` tag** as the primary per-occurrence match key, for two reasons:
1. It's already the value intended for the page title (David's requested naming pattern is meeting name + date).
2. It's human-readable and directly verifiable in support/debugging, unlike an opaque Exchange ID.

`MeetingId` can serve as a secondary/fallback signal if date-string matching proves fragile in practice (e.g. two occurrences captured with slightly different date formatting), but should not be the primary key since it doesn't itself appear anywhere the user can see it.

## Known knock-on effects (must be addressed as part of this build, not after)

1. **`Filter_Existing_Mapping` match key changes** from `SeriesMasterId` alone to `SeriesMasterId` **+ occurrence date**. This is a schema-relevant change — the `RecurringMeetingSectionMap` list currently has one row per series; it will need to support multiple rows per series (one per occurrence date), so `PageSelfUrl`/`PageWebUrl` must be per-occurrence, not per-series-only. `SectionPagesUrl` (the section itself) can remain per-series, since the section is expected to hold all dated pages for that meeting.

2. **Bug 9's "first page in section" workaround is a hard blocker.** `Compose_RealExistingPageId` currently does `first(outputs('Get_Pages_In_Section_Existing_Branch')?['body']?['value'])?['id']` — i.e. whichever page happens to be listed first, with no title or date filtering (`Filter_Pages_By_Title` exists on canvas but its output is not what `Compose_RealExistingPageId` actually reads — worth confirming in Designer whether it's dead code). Once a section holds multiple dated pages, this workaround **will pick the wrong page** silently. This must be replaced with genuine date-based page matching (extract date from existing page titles, filter, match) before per-occurrence pages go live — not treated as a follow-up.

3. **Page title on creation** needs to append the occurrence date. `Compose_SafePageTitle` currently derives the title from `triggerBody()?['text_1']` (meeting title) alone; it needs the extracted date appended, e.g. `{MeetingTitle} — {date}`.

4. **Section vs. page scope confirmed unchanged**: the OneNote *section* continues to represent the whole series (e.g. section "121 Simon / David" holding many dated pages) — no change to section creation/matching logic (`Get_Sections_Recurring` / `Filter_OneNote_Section_Recurring` / `Condition_Section_Exists_Recurring`).

## Not yet answered — needs a build-time decision

- Exact expression to extract the date from `text_3`'s `<title>...</title>` — likely a `substring`/`indexOf` pair since there's no dedicated date field, or confirm whether Flow A could instead pass the date as its own trigger field (`text_5`?) which would be more robust than string-scraping HTML. Worth raising as an option before building the scrape-based approach, since it avoids parsing HTML for a value that's readily available upstream.
- Exact page-title date format to use (e.g. "2 Sep 2026" matches what's already in the HTML title, so reusing that format avoids a second transform).
- Whether matching should be exact-string date match, or tolerant of date format variation between captures (relevant if the scrape approach is used and formatting drifts).

## Related open items this build should double check, not fix in isolation

- Bug 9 first-page-in-section workaround (see above — hard blocker).
- `Filter_Pages_By_Title` — currently present in the existing-branch container but appears unused by `Compose_RealExistingPageId`; confirm whether it's dead code or wire it in as part of the date-matching rebuild.
- Link-format issue (6/8 Aug, reconfirmed 20 Aug): flow returns `PageSelfUrl` (raw OneNote API self-reference) instead of `oneNoteWebUrl` in some outputs, which is unopenable in a browser (`C40001` auth error on click). Not in scope for this amendment but will affect any new page links surfaced via this same mechanism, so worth fixing alongside.

---
*Drafted 20 August 2026, in response to live field-testing session. Evidence sources: run traces `08584143616317951232922356...` (19 Aug capture) and the immediately following run (2 Sep capture), both against "121 Simon / David" series. Cross-reference with `handover-2026-08-16-bug9-closed-workaround-confirmed.md` for the Bug 9 background this amendment depends on.*
