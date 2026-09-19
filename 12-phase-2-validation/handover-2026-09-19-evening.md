# Handover notes — 19 Sep 2026 evening

For the next Claude chat session. All sessions 4-7 are complete. No outstanding build work.

---

## Project summary

Teams → OneNote Meeting Capture. Copilot Studio agent + four Power Automate flows writing meeting pages to OneNote, with a SharePoint mapping list tracking what has been captured.

**Flows (renamed this session):**
- Flow A: `PA - Meeting Capture - A - Resolve Meeting Selection` (`d9d7ccf7`)
- Flow B: `PA - Meeting Capture - B - Resolve OneNote Section` (`ed112c88`)
- Flow C: `PA - Meeting Capture - C - Chat Capture` (`7b295cc2`)
- Flow D: `PA - Meeting Capture - D - Auto Scheduler` (`d5faeba7`)
- SharePoint list: `RecurringMeetingSectionMap` (`186b3c9f-e758-4e85-83d5-685946614a0a`)
- Repo: `Crox2310/Teams-OneNote-Meeting-Capture`, working dir `12-phase-2-validation/`

---

## State at end of 19 Sep evening session

### All flows: published green

### Flow C — current state
Chat capture always runs at the main flow level after FC05c. AllowFallback removed. FC05d deleted. AI insight writes to body via FC05t_Update_Notes (OneNote connector). Chat writes to body via FC15_Update_Chat (OneNote connector). ChatCaptured set once at FC16. Respond fires on FC16 Succeeded only.

Known limitation: content appends below the template slot headings rather than inside them. Slot targeting via raw PATCH is blocked by Sainsbury's DLP policy (HTTP with Microsoft Entra ID connector). This is noted as a future build opportunity.

### Flow B — current state
- Get items Top Count: 500
- FB-F01 one-off prefix: `Evt Mtg -`
- Recurring prefix: `Mtg -` (unchanged)
- UJ3b stale row cleanup added between varPageAction and Condition_IsRecurring
- Existing-branch guard added inside Apply_to_each_Existing_Section

### Flow D — current state
- varOffsetMinutes: 30

### Known-good values
The 18 Sep addendum (`known-good-values-addendum-2026-09-18.md`) is still the active reference for Flow B wipe recovery. It has not been merged into the master reference yet — see below.

---

## Remaining tasks

### Manual cleanup (David to do)
- Delete test OneNote pages created during 19 Sep testing (RFID Scrum of Scrums test pages)

### Session 8 — Docs cleanup (low risk, ~20 min)
1. Merge `known-good-values-addendum-2026-09-18.md` into `known-good-values-master-reference.md`
2. Add new known-good values for Flow C new actions (FC05t_Update_Notes, FC15_Update_Chat, FC16_Set_ChatCaptured)
3. Add new known-good values for Flow B new actions (UJ3b_Filter_Stale_Rows, UJ3b_Delete_Stale_Rows, Guard actions)

### Future build opportunities (no session scheduled)
- Slot targeting for AI insight and chat content — requires IT to unblock HTTP with Microsoft Entra ID connector
- UJ3b test: create a stale row in SharePoint and confirm it is auto-deleted on next Flow B run
- Existing-branch guard test: requires a scenario where mapping row exists but page has been deleted

---

## Acceptance tests (status)
1. New recurring capture: 4-section page created, join link present — NOT YET TESTED end-to-end with live agent
2. Flow D auto-fills Notes + Chat ~30 min after end — NOT YET TESTED with 30 min offset
3. Meeting with no Copilot recap: Chat filled, Notes slot empty — CONFIRMED (RFID Scrum of Scrums 17 Sep)
4. One-off capture: page in One Off Meeting → One-Off Meetings — previously confirmed, not re-tested today
5. Past-date capture: last 50 messages of that occurrence — confirmed by architecture, not re-tested today
6. Existing page (no slots): Flow C falls back to body append — confirmed by design

---

## Key reference values

**One-Off Meetings fixed section URL:**
```
https://www.onenote.com/api/v1.0/myOrganization/siteCollections/b5f8860c-4772-4e8b-b340-e80ba9d490fa/sites/d814850f-59bb-4182-92b7-e25d8c6a0487/notes/sections/1-cbb5e863-2bd7-458f-9944-4fec9d20607b/pages
```

**Notebook key (Master Archive — used in Flow C):**
```
Meeting Notes|$|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Master Archive Folder/Meeting Notes
```

**Notebook key (as used in most Flow B actions):**
```
Meeting Notes|$|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Meeting Notes
```

---

## Known corruption protocol (Flow B)

Value-wipes strike after structural edits. When Flow Checker shows multiple "Value is required" errors:
1. Restore all values from `known-good-values-addendum-2026-09-18.md` (overrides master reference until merged).
2. Save every 5 actions.
3. After restoring, close and reopen the flow, wait 20s, run Flow Checker again before publishing.
4. Check `Condition_IsRecurring` Code view to confirm `varFinalPageDecision_1` = `@outputs('Compose_PageDecision')` not `""`.

Flow C and Flow D have not exhibited the wipe pattern.

---

*Handover written 19 Sep 2026 evening. No next session scheduled.*
