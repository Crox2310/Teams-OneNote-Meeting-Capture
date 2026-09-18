# Weekend build plan — 19–20 September 2026 (project completion)

Companion to `session-2026-09-18-endtime-joinurl-flowd.md` and `known-good-values-addendum-2026-09-18.md`.

---

## Proofs completed 18 Sep (PA - Slot Test flow)

| Technique | Result | Consequence |
|---|---|---|
| UpdatePageContent `replace` on `#data-id` **div** | ❌ `The PATCH target DIV for action replace is not supported` | No replace on divs |
| UpdatePageContent `replace` on `#data-id` **p** | ❌ `The PATCH target P for action replace is not supported` | No replace at all via connector — re-runs may stack blocks (acceptable: Flow D runs once per occurrence) |
| UpdatePageContent `append` into `#data-id` **div** (no position) | ✅ Content lands inside the div, under its heading | **Delivers page ordering** |
| CreatePageInSection into a section **inside a section group** | ✅ 201, stored at `Meeting Notes/One Off Meeting/One-Off Meetings.one` | **Delivers one-off grouping** |
| Section dropdown in connector | ✅ lists sections inside groups | — |
| Get sections in notebook (ST04) lists grouped sections? | ❓ not yet checked | Only matters if lookup-by-name is used for grouped sections |

Note: page content typed into the connector's rich-text editor is escaped as literal text — always pass HTML via expression (as Flow B already does).

### Fixed address — One-Off Meetings section (inside *One Off Meeting* group)

```
https://www.onenote.com/api/v1.0/myOrganization/siteCollections/b5f8860c-4772-4e8b-b340-e80ba9d490fa/sites/d814850f-59bb-4182-92b7-e25d8c6a0487/notes/sections/1-cbb5e863-2bd7-458f-9944-4fec9d20607b/pages
```
Notebook: `Meeting Notes|$|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Master Archive Folder/Meeting Notes`

---

## Scope

### A. Page layout
1. Order: **Meeting Capture → Meeting Notes → Meeting Details → Chat Transcript**.
2. **Meeting Capture** header: title, date, (start–end BST later), **"Join Teams meeting" link** from `Topic.OnlineMeetingUrl` (omitted if blank), "Captured by Meeting Capture agent".
3. **Meeting Details**: invite agenda/description with Teams boilerplate (dial-in, passcode, help, disclaimer) stripped — cut in Flow A when BodyPreview is built (Teams block sits between the long `____` divider lines).
4. Skeleton created by Flow B from Topic C10 `text_3`, **headings only, no placeholder text** (cannot be removed later):
   ```
   <h1>Meeting Capture</h1> …join link…
   <div data-id="notes"><h2>Meeting Notes</h2></div>
   <div data-id="details"><h2>Meeting Details</h2> …invite body… </div>
   <div data-id="chat"><h2>Chat Transcript</h2></div>
   ```
5. Flow C fills slots with **target `#notes` / `#chat`, action `append`, no position**. Existing pages (no slots) → fall back to `body` append via configure-run-after.
6. Check Flow B `UpdatedAppend` path (re-capture of existing page) doesn't append the whole skeleton again.

### B. Chat on every run
1. Flow C: insight (if any) → `#notes`; **always** chat chain → `#chat`; set ChatCaptured once at end.
2. Chat = **last 50 messages of that occurrence**, anchored to EndTime (so past-date captures get the right 50), displayed oldest→newest. Candidate: `$top=50&$orderby=lastModifiedDateTime desc&$filter=lastModifiedDateTime lt <EndTime+15m>` — prove in scratch first (Teams connector HTTP).
3. AllowFallback gate kept only for: no insight **and** AllowFallback=false → defer.
4. Fix FC14 notebookKey path mismatch.

### C. Section organisation
1. **One-offs**: Flow B one-off path always creates the page in the fixed **One-Off Meetings** section above, title `<Meeting> - <d MMM yyyy>`. Removes most of the D2 branch (Get Sections D2 / Filter / Condition / Create Section D2 / 4 SetVariables).
2. **Recurring**: rename prefix `Mtg - ` → `Rec - ` (same 6 chars; truncation maths unchanged) in `Compose_SafeSectionName` and `Compose_SafeSectionName_ExistingBranch`; rename existing sections in OneNote at the same time (rename keeps IDs).
3. *(Later)* Recurring group: requires Flow B to reuse the series' previous `SectionPagesUrl` instead of name lookup.

### D. Reliability / close-out
1. Flow C Respond placement — Flow D must see failures.
2. UJ3b stale-row cleanup — failed Flow B runs leave skeleton rows that block recapture.
3. AllowFallback cut-off rule (suggest 60 min after end).
4. Merge 18 Sep addendum into canonical known-good-values docs; document *Apply to each Existing Section* setters; add new one-off path and page template.
5. Tidy: dead FA10–FA12 loop; Mid-Year all-day entry through FA09B; unused `Compose_SafeSectionName_D2`; unused FA43 `endtime` + C2 binding; delete PA - Slot Test flow and test pages.

---

## Decisions to confirm at start
1. Header contents (A2) — add attendees / organiser / start time?
2. Cut-off rule value.
3. Existing one-off sections — leave, or move pages into One-Off Meetings by hand.

---

## Session plan

| # | Session | Model | Output |
|---|---|---|---|
| 0 | Pre-flight: Flow Checker sweep A/B/C/D; Code-view snapshots of Flow B + C | Sonnet | Known-good "before" |
| 1 | Design pass: final template HTML, Flow C branch structure, cut-off rule | Opus (short) | Spec |
| 2 | Proof: chat `lastModifiedDateTime` filter via Teams connector HTTP | Sonnet | Go/no-go |
| 3 | Page template: Topic C10 `text_3` skeleton + Flow A boilerplate strip | Sonnet | New layout |
| 4 | Flow C restructure: slot appends, chat every run, last 50, Respond fix, FC14 path | Sonnet | Full page auto-filled |
| 5 | Flow B one-off path → fixed One-Off Meetings section; `Rec - ` prefix + OneNote renames | Sonnet | Grouped sections |
| 6 | UJ3b + cut-off rule | Sonnet | Self-healing |
| 7 | E2E tests + docs merge + push | Sonnet | Done |

**Rule for sessions 3–6:** every delete/move risks the propagation wipe — after each structural edit: save → close/reopen → Code-view check → publish. Agents call *published* flows; test from a new Teams chat.

---

## Acceptance tests
1. New recurring capture from Teams: 4 sections in order, join link at top, row has ISO EndTime + JoinUrl, lands in `Rec -` section.
2. Flow D fills Notes + Chat ~5–15 min after end; ChatCaptured ✓.
3. Meeting with no Copilot recap: Chat filled, Notes shows "No recap available".
4. One-off capture: page created in One Off Meeting → One-Off Meetings; no new section.
5. Past-date capture: last 50 messages *of that occurrence*.
6. Existing (pre-template) page: Flow C falls back to body append without failing.
