# Redesign Specification — v5 UX Rebuild
**Date:** 2026-06-27
**Author:** Claude (session with David Croxson)
**Upload to:** `12-phase-2-validation/design-v5-ux-rebuild-2026-06-27.md`

---

## Why we are redesigning

The v4 Topic required users to type a meeting name, which Flow A then searched for using `triggerBody()?['text']`. This field is never reliably populated when Flow A is called as a Copilot Studio agent action — the parameters arrive as named fields (`UserSearchText`, `InSelectedNumber` etc), not as `text`. This mismatch caused FA09A to always receive an empty search string and return no matches, making UJ2 untestable.

The new design eliminates the search entirely. Instead Flow A returns the full day's meeting list, the user browses and selects by number, and the agent proceeds directly to Flow B.

---

## New User Experience

```
User: "capture meeting notes"

Agent: Here are your meetings for today (Sat 27 Jun):

1. Capture (07:00)
2. Ace Meeting Morning (17:00)
3. Ace Meeting Afternoon (17:30)
4. QWE Stand Up (18:30)
5. TTT meeting (19:30)
6. QWE Meeting (21:00)

Type a number to capture, P for previous day,
N for next day, or a date (e.g. 28 Jun) to jump.

User: "2"

Agent: Your meeting Ace Meeting Morning has been found.
Creating your OneNote page now...

Agent: Great news! Your meeting notes for Ace Meeting Morning
have been saved to OneNote. Here's your page link: [link]
```

---

## Flow A — Changes Required

### Inputs (simplified)
| Field | Type | Purpose |
|---|---|---|
| `text` | String | DateContext — date to fetch meetings for (default "today") |
| `text_1` | String | InSelectedNumber — blank on first call, number on selection call |

`UserSearchText`, `OriginalUserSearchText`, `MaxCandidates` are no longer needed and should be removed from the trigger schema.

### Actions to remove
- `FA09A Filter CandidatesByTitle` — entire action deleted
- All `triggerBody()?['text']` / `triggerBody()?['text_2']` search text references
- `FA03A DEBUG RawInputs` — can be removed once stable
- `FA01 Init varUserSearchText` — no longer needed
- `FA03 Init varOriginalUserSearchText` — no longer needed
- `FA05 Init varMaxCandidates` — no longer needed

### Actions to keep unchanged
- **FA06 Compose StartOfDayUtc** — keep, date range calculation unchanged
- **FA07 Compose EndOfDayUtc** — keep
- **FA08 Get calendar events** — keep, connector call unchanged
- **FA08A DEBUG RawConnectorOutput** — keep for now
- **FA09 RAW CandidateArray** — keep (rename optional: `FA09_AllMeetingsForDay`)
- **FA10 Initialize varCandidates** — keep
- **FA11 Apply to each Candidates** — keep, now iterates ALL meetings (no pre-filter)
- **FA12 Append to array varCandidates** — keep
- **FA13 Compose count** — keep, counts all meetings
- **FA27 NO_MATCH condition** — keep (handles genuinely empty calendar day)
- **FA28–FA43 single-match resolution** — keep entirely unchanged
- **FA43 Respond to agent** — keep, output schema unchanged

### Actions to change

**FA02 Init varInSelectedNumber** — keep, but verify it reads `triggerBody()?['text_1']` (was previously `text_1` = InSelectedNumber).

**FA04 Init varDateContext** — change to read `triggerBody()?['text']` (DateContext is now the first parameter, previously `text_3`).

**FA14 Compose CandidateList** — change format to include start times, numbered, sorted by start time. New format:
```
1. Meeting Name (HH:MM)
2. Meeting Name (HH:MM)
```
Time should be extracted from the `start` field and converted to local time (BST = UTC+1 in summer). Expression to extract time portion:
```
formatDateTime(item()?['start'], 'HH:mm')
```
Full FA14 Compose expression (built inside FA11 Apply to each, appended per item):
```
concat(string(variables('varCandidateIndex')), '. ', item()?['subject'], ' (', formatDateTime(item()?['start'], 'HH:mm'), ')')
```
Note: a counter variable `varCandidateIndex` will need to be added and incremented inside FA11.

### Outputs (unchanged)
The seven-field output schema stays identical:

| Output field | Content |
|---|---|
| `status` | OK / NO_MATCH / ERROR |
| `matchcount` | Number of meetings found |
| `candidatelist` | Formatted numbered list with times |
| `meetingtitle` | Selected meeting title (on second call) |
| `calendareventid` | Selected meeting calendar ID (on second call) |
| `isrecurring` | true/false string |
| `seriesmasterid` | Series master ID or empty string |

Flow B contract is completely untouched.

---

## Topic — Changes Required

### Trigger phrases (keep existing)
`capture meeting notes`, `capture notes for`, `record meeting notes`, `create meeting notes`, `capture`

### New node structure

```
Trigger
  ↓
C1_Set_DateContext
  Set variable: Topic.DateContext = "today"
  ↓
C2_Call_FlowA_List
  Call Flow A: text = Topic.DateContext, text_1 = ""
  ↓
C3_Check_MatchCount
  Topic.MatchCount is equal to "0"
  → True:  C3A_NoMeetings_Message
           "I couldn't find any meetings for that day.
            Type a date (e.g. 28 Jun), P or N to navigate."
           → loop back to C5_Ask_Selection
  → False: continue
  ↓
C4_Display_MeetingList
  Message: "Here are your meetings for {Topic.DateContext}:
  
  {Topic.CandidateList}
  
  Type a number to capture, P for previous day,
  N for next day, or a date (e.g. 28 Jun) to jump."
  ↓
C5_Ask_Selection
  Question node — captures user response into Topic.UserInput
  ↓
C6_Check_Input  [Condition branch — four-way]
  ├── Topic.UserInput is equal to "P" (case-insensitive)
  │     C6A: Topic.DateContext = addDays(Topic.DateContext, -1)
  │     → loop back to C2_Call_FlowA_List
  │
  ├── Topic.UserInput is equal to "N" (case-insensitive)
  │     C6B: Topic.DateContext = addDays(Topic.DateContext, 1)
  │     → loop back to C2_Call_FlowA_List
  │
  ├── Topic.UserInput is a valid date (check using IsMatch or similar)
  │     C6C: Topic.DateContext = Topic.UserInput (parsed date string)
  │     → loop back to C2_Call_FlowA_List
  │
  └── All other conditions (assumed to be a number)
        C7_Call_FlowA_Selection
        Call Flow A: text = Topic.DateContext, text_1 = Topic.UserInput
          ↓
        C8_Confirm_Meeting
        "Your meeting {Topic.MeetingTitle} has been found.
         Creating your OneNote page now..."
          ↓
        C9_Call_FlowB
        Call Flow B — inputs and outputs unchanged from current v4
          ↓
        C10_Check_OutStatus
        Topic.OutStatus is equal to "OK"
        ├── True:  C10A_Success
        │          "Great news! Your meeting notes for
        │           {Topic.MeetingTitle} have been saved to OneNote.
        │           Here's your page link: {Topic.OutCreatedPageLink}"
        └── False: C10B_Error
                   "I'm sorry, something went wrong saving your
                    meeting notes. Please try again."
```

### Date navigation implementation

| Input | Action |
|---|---|
| P | `addDays(Topic.DateContext, -1)` |
| N | `addDays(Topic.DateContext, 1)` |
| Date string (e.g. "28 Jun") | Set `Topic.DateContext` to parsed date, pass to Flow A |
| Number (e.g. "2") | Pass as `InSelectedNumber` to Flow A second call |

DateContext format passed to Flow A: `yyyy-MM-dd` string. Flow A's FA06/FA07 convert this to UTC start/end of day range for the calendar API call.

### Variables needed

| Variable | Type | Purpose |
|---|---|---|
| `Topic.DateContext` | String | Current date being browsed (default "today") |
| `Topic.UserInput` | String | Raw user input (number, P, N, or date) |
| `Topic.MatchCount` | String | From Flow A output |
| `Topic.CandidateList` | String | From Flow A output — pre-formatted with times |
| `Topic.MeetingTitle` | String | From Flow A output |
| `Topic.CalendarEventId` | String | From Flow A output |
| `Topic.IsRecurring` | String | From Flow A output |
| `Topic.SeriesMasterId` | String | From Flow A output |
| `Topic.OutStatus` | String | From Flow B output |
| `Topic.OutCreatedPageLink` | String | From Flow B output |

---

## Flow B — No changes

Flow B (`PA - Resolve OneNote Meeting Section - v2 Clean Build`) is confirmed working end-to-end (UJ1 passing, re-baselined 2026-06-26). Its trigger contract, internal logic, and all outputs are completely unchanged.

Flow B trigger inputs (from Topic, unchanged):

| Flow B field | Topic source |
|---|---|
| `text` (IsRecurring) | `Topic.IsRecurring` |
| `text_1` (MeetingTitle) | `Topic.MeetingTitle` |
| `text_2` (SeriesMaster) | `Topic.SeriesMasterId` |
| `text_3` (PageHtml) | `Concatenate("<h1>", Topic.MeetingTitle, "</h1><p>Meeting notes captured via Teams OneNote Meeting Capture agent.</p>")` |
| `text_4` (MeetingId) | `Topic.CalendarEventId` |

---

## What we are NOT changing

- Flow B entirely
- Flow A's FA08 Outlook calendar connector call
- Flow A's FA10–FA13 array build loop
- Flow A's FA27 NO_MATCH branch
- Flow A's FA28–FA43B single-match resolution and selection path
- Flow A's seven-field output schema
- Flow A's FA43 Respond to agent action
- The OneNote section/page creation logic in Flow B
- SharePoint RecurringMeetingSectionMap integration

---

## Build sequence (recommended)

1. **Flow A first**
   - Remove FA09A, FA01, FA03, FA05 actions
   - Update FA02 to read `text_1`, FA04 to read `text`
   - Update FA14 CandidateList format to include numbered list with times
   - Add `varCandidateIndex` counter variable, increment in FA11
   - Save draft → Flow checker → Publish
   - Verify in Activity: CandidateList output shows numbered list with times

2. **Topic second**
   - Rebuild node structure per design above
   - Wire C2 → Flow A with DateContext
   - Build C6 four-way branch (P / N / date / number)
   - Wire C7 → Flow A with selection number
   - Wire C9 → Flow B (unchanged from current C8B/C10)
   - Save → Topic checker → 0 errors → Publish

3. **Connection refresh**
   - Settings → Connection Settings
   - Refresh in order: Find Outlook Meeting Details → Resolve OneNote Meeting Section → Resolve Meeting Selection (last)

4. **Test in Copilot Studio test panel**
   - Type "capture meeting notes"
   - Verify meeting list displays with times
   - Type a number → verify OneNote page created
   - Type P / N → verify list refreshes for different day
   - Type a date → verify list refreshes for that date

5. **Teams test**
   - Fresh thread (or Allow consent if prompted)
   - Full end-to-end UJ1-equivalent then UJ2 navigation

---

## Open items carried forward (not blocking this redesign)

These were flagged in the living audit and remain unresolved but are not on the critical path:

- FA12 IsRecurring derivation — 🟡 superseded by FA28A seriesMasterId check in most paths
- `Condition Section Exists OneOff` — 🟡 `greater(...) equal to @true` pattern not re-expanded
- `Set varPageAction Created` / `Set varPageAction UpdatedAppend` — 🟡 not confirmed
- `Set varOutputPageLink Created OneOff` — 🟡 not confirmed
- `Compose IgnoreSeriesMasterId` — 🟡 literal `''`, low priority
- `runAfter` casing (`"SUCCEEDED"` vs `"Succeeded"`) — unresolved, not causing live failures
- `outpageaction` and `outpagedecision` returned `""` in UJ1 run — flagged, not blocking

---

## Session context (2026-06-27)

This redesign was triggered after extensive debugging of the v4 search-based UX revealed a fundamental mismatch: Copilot Studio agent flows receive parameters as named trigger fields, not via `triggerBody()?['text']`. FA09A's filter expression was using the wrong field, causing it to always receive an empty search string and return no matches regardless of calendar data.

Rather than patch the search approach further, the decision was made to redesign around a browse-and-select UX, which is simpler for the user, simpler architecturally, and eliminates the entire search/filter layer that was causing problems.

UJ1 remains confirmed passing (re-baselined 2026-06-26 at 07:33 UTC, TTT meeting, OneNote page and link confirmed).
