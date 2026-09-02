# Known Bugs

## Bug: `Condition Is Genuine Existing Page` misclassifies rows with empty `PageSelfUrl`

**Flow:** PA - Resolve OneNote Meeting Section (Flow B)
**Action:** `Condition Is Genuine Existing Page`
**Found:** 2 Sep 2026

**Symptom:** Flow C fails at `FC13_Compose_PageId` with:
> `The template language function 'split' expects its first parameter to be of type string. The provided value is of type 'Null'.`

**Root cause:**
1. `Compose_PageDecision` classifies a mapping row as `PAGE_NOT_FOUND` whenever that row's `PageSelfUrl` field is empty — even if the row itself genuinely exists and matches on `SeriesMasterId` + `OccurrenceDate`.
2. This routes into the `PAGE_NOT_FOUND` branch, which searches OneNote by page title (`S1_Filter_Pages_By_Title_PreCreate`) and eventually reaches `Condition Is Genuine Existing Page`.
3. That condition's logic currently evaluates `contains(...)` as `true` — but it evaluates to **True even when `varOutputPageSelfUrl` is an empty string** (confirmed via run output: `varOutputPageSelfUrl` = `""`, condition still took the True branch).
4. Because the condition wrongly says "yes, this is a genuine existing page," it proceeds into the True-branch logic (`Get Sections Existing Branch` → `Filter Existing Section By Name` → `Apply to each Existing Section`) — but since there was no real found page to begin with, `varOutputPageSelfUrl` never gets populated with an actual value. It stays empty.
5. The mapping row in SharePoint (`RecurringMeetingSectionMap`) never gets its `PageSelfUrl`/`PageWebUrl` fields written, even though Flow B reports success.
6. Flow C's `FC03_Compose_PageSelfUrl` reads this empty field from the row, and `FC13_Compose_PageId`'s `split(null, '/')` call fails because there's nothing to split.

**What needs fixing:** The condition expression for `Condition Is Genuine Existing Page` needs to actually check whether `varOutputPageSelfUrl` (or whatever it's testing) is non-empty/non-null, not just evaluate a `contains(...)` check that happens to return true on empty input. Needs inspection of the exact expression in that condition action to identify why an empty string satisfies it.

**Affected test case:** "Weekly Check in - SCT Product/Business", `SeriesMasterId eq 'AAMkAGY0OGU4Mzk5LWQ4NTYtNDU4MS1hY2YyLTQxOWYwZjhiMWM1ZQBGAAAAAADWkXK1vW2mQ4SwNGpyD7SzBwB8mPnOPkRmT5-MxNoNopoPAAAAAAENAAB8mPnOPkRmT5-MxNoNopoPAAYqDe0VAAA='`, `OccurrenceDate eq '2026-09-02'` — row ID present in `RecurringMeetingSectionMap`, `PageSelfUrl`/`PageWebUrl` currently empty despite the OneNote page genuinely existing.

**Also worth checking once fixed:** whether this same logic bug affects the *one-off* (non-recurring) branch of Flow B — there's likely a parallel `Condition Is Genuine Existing Page`-equivalent for one-off meetings that may have the identical flaw.

---

## Fixed this session (2 Sep 2026)

- **Flow A `OnlineMeetingUrl` extraction bug** — Graph's structured `onlineMeeting.joinUrl` field is not populated for certain invite types (e.g. `teams.microsoft.com/meet/...` style links). Added a `BodyPreview` HTML fallback extraction (`href="https://teams.microsoft.com/...`) in both the "user selected a specific meeting" and "single auto-resolved match" branches. Built and confirmed working on live data.
- **Flow B not writing `JoinUrl` through to SharePoint** — Flow B's trigger never accepted an `OnlineMeetingUrl` input, so it was never written to the `RecurringMeetingSectionMap` mapping row on create or update. Added `text_6` (`OnlineMeetingUrl`) to Flow B's trigger schema, wired it into both create paths (recurring + one-off) and both update paths (recurring + one-off), and updated the Copilot Studio Topic's `C10_Call_FlowB_Create_Page` binding to pass `Topic.OnlineMeetingUrl` through as `text_6`. Built and confirmed working on live data.
- **Flow C transcript-capture branch (FC15–FC21) removed** — blocked by the `GraphAccessToTranscriptsDisabled` tenant-level Graph API setting (separate from `OnlineMeetingTranscript.Read.All` permission consent). Removed the branch entirely from Flow C so it no longer blocks the chat-capture path (FC01–FC22); FC12 updated to remove its now-broken reference to the deleted `FC20_Compose_TranscriptBlock`/`FC21_Compose_TranscriptBlock` outputs. To be rebuilt once tenant admin access is confirmed and the transcript block is lifted.
