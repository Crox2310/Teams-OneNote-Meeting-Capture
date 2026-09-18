# Session log — 18 September 2026

## Outcome

Automatic post-meeting capture is **working end to end** for recurring meetings captured via the Topic (Teams or Test pane):

Topic → Flow A (candidates incl. End + JoinUrl) → Flow B (page + mapping row with **EndTime** and **JoinUrl**) → Flow D (polls every 10 min, fires at EndTime + 5 min) → Flow C (AI Insight append, ChatCaptured = true).

First full automatic pass: STDA 17 Sep (row 364), AI Insight rendered correctly in OneNote. Subsequently verified from Teams with fresh captures.

---

## What was built

### 1. EndTime plumbing (replaces calendar lookup in Flow D)
- **Flow A**: V3 calendar view returns `start`/`end` as flat strings plus `endWithTimeZone` (ISO 8601 with `+00:00`). This is the value used.
  - `FA28C_Compose_OutEndTime` (single-match path) + FA43 output `endtime` (currently unused by the Topic — see agent tool note below).
  - `FA12B_Select_Candidates` key `End` = `concat('UTC|', coalesce(item()?['endWithTimeZone'], ''))`.
- **Topic C6D**: `Topic.EndTime = Text(Index(ParseJSON(Topic.CandidatesJson), n).End)`.
- **Flow B**: new optional trigger input `text_6` (EndTime); `Create_Mapping_Item_Recurring` writes `item/EndTime = replace(coalesce(triggerBody()?['text_6'], ''), 'UTC|', '')`.

**Why the `UTC|` prefix:** Power Fx in the Topic silently re-parses any ISO date string passed through `Text()` (and `DateTimeValue()`), re-formatting it as US `MM/dd/yyyy HH:mm:ss` in the Copilot Studio server timezone (observed US Pacific, −7h) with **no timezone marker**. That made Flow D fire ~7h early. Prefixing makes the value non-date so Power Fx passes it through untouched; Flow B strips the prefix. `DateTimeFormat.UTC` approach was tried and did **not** fix it.

### 2. JoinUrl plumbing
- **Flow A**: V3 events have **no `onlineMeeting` field** — join URL always comes from the body. Newer invites put a short `/meet/…?p=` link first; Graph `joinWebUrl` lookup needs the `/l/meetup-join/` link.
  - `FA29D` anchor changed to `href="https://teams.microsoft.com/l/meetup-join/`.
  - `FA12B` `OnlineMeetingUrl` (was hardcoded `""`) now extracts the meetup-join href from the full body.
- **Topic C10**: `text_7 = Topic.OnlineMeetingUrl`.
- **Flow B**: new optional trigger input `text_7` (JoinUrl); `Create_Mapping_Item_Recurring` writes `item/JoinUrl = trim(coalesce(triggerBody()?['text_7'], ''))`.

### 3. Flow D redesign (PA - Auto Capture Scheduler)
- Deleted FD05 / FD05b / FD06a / FD06b (calendar lookup, matching).
- `FD04` filter adds EndTime non-empty guard.
- `FD06` now loops `body('FD04_—_Filter_Uncaptured_Rows')` — **it previously looped calendar events while reading SharePoint fields, the cause of the 17 Sep misfire.**
- `FD06c` = `ticks(utcNow()) >= ticks(addMinutes(items('FD06_—_For_Each_Uncaptured_Row')?['EndTime'], variables('varOffsetMinutes')))` directly containing `FD06d`.

### 4. Flow C fixes
- `FC01` filter trigger references had been blanked when the trigger was swapped manual → PowerAppV2 (17 Sep). Restored.
- `FC02` now `trim()`s JoinUrl.
- Confirmed: `GetOnlineMeeting` / AI Insights work for meetings David **attends**, not only organises.

---

## Incidents

- **Flow A value-wipe** (session start): FA33A / FA34A again — restored from reference.
- **Flow D propagation wipe**: deleting FD06b/FD06a/FD05b/FD05 blanked FD06d inputs — restored.
- **Flow B 30-action value-wipe** (+ 5 SharePoint connection errors after session loss): restored twice from master reference. Connections re-pointed via Change connection; Test pane then required a one-time connection consent (Allow).
- **Agent publish blocked** by `InvalidPropertyPath` on the **agent-level tool registration** of Flow A (tool page Inputs showed (!), refresh greyed). Resolved by setting that agent tool to **Disabled**. The Topic still calls Flow A via its C2 node; agent instructions forbid direct tool calls anyway.
- **Skeleton mapping rows**: failed Flow B runs (during the wipe) left rows with blank SectionPagesUrl. Subsequent captures took the *Mapping Exists* branch and failed with empty sectionId. Fixed by deleting the rows. Confirms **UJ3b** is a real gap.

---

## Lessons

1. **Power Fx and dates**: never pass ISO date strings through `Text()` in a Topic if the string must survive intact — prefix or otherwise make them non-date.
2. **Published vs saved**: agents call the *published* flow version; agent changes need a new Teams chat to take effect.
3. **Replacing a trigger blanks every dynamic reference to it** — sweep all trigger readers after any trigger swap.
4. **Flow Checker misses some blanks** — verify key actions in Code view.
5. **Agent-level tool registrations can go stale independently of the Topic** — if publish fails with InvalidPropertyPath on a flow tool, check the Tools tab entry.

---

## Backlog (priority order)

1. **Flow C Respond placement** — Respond runs even when capture fails, so Flow D sees success.
2. **UJ3b stale-row cleanup** — failed Flow B runs leave skeleton rows that block recapture of that occurrence.
3. **AllowFallback cut-off rule** — Flow D passes `"true"`; an early fire takes chat fallback and never retries the AI Insight. Decide once real insight-latency data exists (Opus-level design).
4. **Merge the 18 Sep addendum into the canonical known-good-values docs** and document the three existing-branch SetVariables (inside *Apply to each Existing Section*) — not yet captured.
5. Tidy-up: dead FA10–FA12 loop in Flow A; FA09B lets the all-day *Mid-Year Link Conversation Window* through; FC14 notebookKey path mismatch; unused `Compose_SafeSectionName_D2`; unused FA43 `endtime` output + C2 binding.
6. Agent-level Flow A tool is Disabled — leave off, or remove/re-add cleanly if ever needed.
