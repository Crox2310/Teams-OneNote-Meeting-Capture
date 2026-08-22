# New feature requests and bugs — logged 22 August 2026 (end of day)

Items raised by David at close of day. To be investigated and prioritised at the start of the next session alongside the existing backlog.

---

## FR-01 — Candidate list ordering: chronological by meeting time

**Request:** when the agent displays the candidate list of meetings for a given day, what determines the order? David's preference is chronological order by meeting start time, so the list reads naturally top-to-bottom through the day.

**Current behaviour:** order is determined by how the Microsoft Graph calendar API (`FA08 Get calendar view of events`) returns events. Graph typically returns events in start-time order, but this isn't guaranteed and may vary.

**Investigation needed:** check `FA08`'s raw output order in a live Activity trace, then confirm whether the candidate list Compose (`FA14`) preserves that order or reorders it. If Graph returns chronologically but the list is built in a different order, a sort step may be needed. If Graph already returns chronologically, no change is needed.

**Complexity:** Low-Medium. If a sort is needed, Power Automate's `sort()` function can be applied to `varCandidates` before building the list.

---

## FR-02 — Filter out holiday/leave entries from candidate list

**Request:** some colleagues send calendar blocks for holidays, annual leave, or similar non-meeting events (e.g. "David Croxson - Holiday", "Sarra Beattie - A/L", "Team Leave"). These appear in the candidate list but are not real meetings to capture. David would like these filtered out automatically.

**Filter criteria (examples given):**
- Title contains "holiday" (case-insensitive)
- Title contains "leave" (case-insensitive)
- Title contains "AL" or "A/L" (case-insensitive)

**Current behaviour:** all calendar events in the day's time window are included in the candidate list regardless of title.

**Implementation approach:** add a filter step after `FA08 Get calendar view of events` (or inside `FA11 Apply to each Candidates`) that excludes events whose subject matches any of the above patterns. Power Automate's `contains()` with `toLower()` would handle case-insensitive matching.

**Complexity:** Low. A filter array action or a condition inside the existing `Apply to each` loop.

**Note:** worth checking whether there are other common patterns David wants excluded (e.g. "OOO", "out of office", "bank holiday") before building, so the filter list is comprehensive first time.

---

## FR-03 — Shorten the OneNote page link returned by the agent

**Request:** the link returned in the agent's success message is very long (a full SharePoint URL with encoded path components). David would like a shorter, more meaningful link.

**Current behaviour:** `OutCreatedPageLink` returns the full `oneNoteWebUrl` from the OneNote API, e.g.:
```
https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Meeting%20Notes?wd=target%28Mtg%20-%20121%20Simon%20...
```

**Options to investigate:**
1. **SharePoint short links** — SharePoint has a "Copy link" feature that generates shorter `aka.ms`-style or tenant-specific short URLs. Whether these are accessible via API needs checking.
2. **OneNote deep link** — the OneNote API also returns a `oneNoteClientUrl` (opens in the OneNote app) which may be shorter/more meaningful than the web URL.
3. **Custom display text** — rather than shortening the URL itself, display it as a clickable hyperlink with meaningful text (e.g. "Open in OneNote") using Teams Adaptive Card formatting. This may be the most practical option without requiring an additional API call.

**Complexity:** Medium. Depends on which approach is chosen.

---

## BUG-01 — Second occurrence of recurring series overwrites first page

**Reported:** 22 August 2026, evening.

**Symptom:** captured a recurring meeting successfully (first occurrence). Captured the next occurrence of the same series (following week). The second capture appeared to overwrite or merge with the first page rather than creating a separate dated page.

**Expected behaviour:** each occurrence should create its own dated page (e.g. `121 Simon / David - 22 Aug 2026` and `121 Simon / David - 29 Aug 2026`) as separate siblings in the same section.

**Likely root causes (to confirm from Activity trace):**
1. **Corruption** — `Set_varOutStatus` and other SetVariable actions were found corrupted at close of day (19 errors). If `Filter_Existing_Mapping` or related actions were also corrupted, the flow may have taken the wrong path.
2. **OccurrenceDate mismatch** — if `text_5` (OccurrenceDate) is not being sent correctly for the second occurrence, `Filter_Existing_Mapping` may match the first occurrence's mapping row, routing the flow to the existing-page update path instead of the create-new-page path.
3. **`Filter_Pages_By_Title` matching wrong page** — if both occurrences have the same page title (e.g. if FB-05's dated title didn't apply correctly to the first page), the filter may match the first page when processing the second occurrence.

**Next steps:** pull Activity trace from the failing run. Check `Get_items` output, `Filter_Existing_Mapping` output, `Compose_PageDecision`, and `Filter_Pages_By_Title` output in that order. This is the **highest priority investigation item for next session** — it's a real, observed regression on the core feature.

---

## Priority for next session (updated)

1. **BUG-01** — recurring series second occurrence overwriting first page. Highest priority, investigate from Activity trace first.
2. **Recurring meeting capture errors** — general investigation (may overlap with BUG-01).
3. **FR-02** — holiday/leave filter. Low complexity, high practical value.
4. **FR-01** — chronological ordering. Low complexity, worth confirming current behaviour first.
5. **FR-03** — link shortening. Medium complexity, investigate options.
6. **UJ3b, UJ4c** — as previously planned.
