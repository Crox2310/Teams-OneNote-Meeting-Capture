# Phase 2 — Product vision and scope

**Written:** 22 August 2026, end of day.
**Status:** not yet started. Phase 1 (meeting capture to OneNote) must be fully stable before Phase 2 begins.

---

## Overview

Phase 2 extends the Teams → OneNote Meeting Capture agent in two directions:

1. **Post-meeting note ingestion** — after a meeting has run, automatically capture notes, transcriptions, actions, and other meeting artefacts into the existing OneNote page for that meeting, without overwriting anything already there.

2. **Pre-day preparation** — automatically run the capture agent before the day starts, creating OneNote pages for all of the day's meetings in advance, ready to receive notes.

---

## Feature 1: Post-meeting note ingestion

### What it does
After a meeting completes, the agent:
- Retrieves meeting artefacts from Teams: transcription, meeting notes, action items
- Finds the existing OneNote page for that meeting (using the same mapping table logic as Phase 1)
- Appends the artefacts to the page in a structured, clearly labelled format
- **Never overwrites existing content** — all additions are appended below existing human-entered notes, using the same safe-append pattern already established in Phase 1's recapture flow

### Source artefacts to consider
- **Teams meeting transcription** — available via Microsoft Graph (`/communications/callRecords` or `/onlineMeetings/{id}/transcripts`). May require additional Graph permissions.
- **Teams meeting notes** — if the meeting used the built-in Teams meeting notes feature, these may be accessible via Graph.
- **Action items** — if actions were captured in Teams (e.g. via Loop components or the Tasks integration), these could be retrieved and appended as a structured list.
- **Meeting recording** — a link to the recording (if one exists) could be appended as a reference.

### Key design constraints
- **Non-destructive:** existing OneNote content must never be overwritten or deleted. The safe-append pattern from Phase 1 (`Compose_UpdateHtmlFragment`) is the right foundation.
- **Idempotent:** running the agent twice after the same meeting should not duplicate content. Needs a mechanism to detect whether post-meeting notes have already been appended (e.g. a flag in the SharePoint mapping row, or a content check on the page).
- **Triggered appropriately:** the agent should fire after the meeting ends, not during. Options: scheduled trigger (e.g. X minutes after the meeting's end time), Teams meeting-end event (if available via Graph webhooks), or manual trigger by the user.

### Open questions
- Which Graph API endpoints are available in the Sainsbury's tenant for transcriptions and meeting notes? Some require specific admin consent.
- How long after a meeting ends are transcripts available? There's typically a processing delay.
- Should this be a separate flow triggered automatically, or an extension of the existing agent triggered manually by the user?

---

## Feature 2: Pre-day preparation

### What it does
Before the day starts (e.g. at 7am), the agent automatically:
- Retrieves all meetings for the coming day from the user's calendar
- For each meeting, creates an OneNote page (if one doesn't already exist) with the meeting details pre-populated
- Skips meetings that already have a page (idempotent — safe to run multiple times)
- Skips non-meeting entries (holidays, leave, OOO — FR-02 filter logic applies here too)

### Key design considerations
- **Trigger:** a scheduled Power Automate flow (recurrence trigger, e.g. daily at 07:00) rather than a user-initiated agent conversation
- **No user interaction required:** unlike Phase 1, this runs fully automatically. No candidate list, no selection prompt.
- **Multi-meeting handling:** Phase 1 processes one meeting at a time (user selects from a list). Phase 2's pre-day prep needs to process all meetings in a loop — a Foreach over the day's calendar events.
- **Conflict with Phase 1:** if the pre-day prep creates a page for a meeting, and the user later runs the Phase 1 agent for the same meeting, Phase 1 must correctly detect the existing page and append rather than create a duplicate. The existing mapping table + `Filter_Existing_Mapping` logic should handle this, but needs verifying.
- **Notification:** after pre-day prep completes, send the user a Teams message summarising which pages were created (similar to the current success message).

### Open questions
- Should pre-day prep run for all calendar events, or only meetings the user is hosting/organising?
- What time should it trigger? 07:00 is a reasonable default but should be configurable.
- Should it cover just the next day, or the next N days (e.g. the full working week ahead)?

---

## Architecture implications for Phase 2

Phase 2 introduces new complexity that may warrant revisiting the Phase 1 architecture:

- **A third flow** — the pre-day prep is a scheduled, automated flow with no agent interaction. It shares logic with Flow B (OneNote page creation) but has a different trigger and iteration model. Reusing Flow B directly may be possible but could also create a tangled dependency. A cleaner option may be a dedicated `Flow C — Pre-Day Prep` that calls Flow B for each meeting.

- **A fourth flow** — post-meeting note ingestion is similarly distinct: triggered by meeting end, fetches different source data (transcripts vs. calendar details), and needs to find an existing page rather than create one. Could reuse Flow B's append logic but likely warrants its own flow.

- **The mapping table becomes more valuable** — the SharePoint mapping table is essential for Phase 2: it's how the post-meeting flow finds the right OneNote page for a given meeting. This changes the data retention question — rows need to be kept at least until post-meeting notes have been successfully appended, not just until the meeting has been captured.

- **Graph permissions** — transcription access requires additional Microsoft Graph permissions (`OnlineMeetings.Read` or similar) that may need IT/admin approval in the Sainsbury's tenant.

---

## Suggested sequencing

1. Complete Phase 1 stabilisation (BUG-01, remaining FRs, strategic review)
2. Conduct the strategic review — architecture decisions made here may affect Phase 2 design
3. Scope Phase 2 Feature 2 (pre-day prep) first — simpler trigger model, reuses existing Flow B logic, high daily-usage value
4. Scope Phase 2 Feature 1 (post-meeting notes) second — more complex, depends on Graph permissions and transcript availability

---

## Why this matters

Phase 1 solves the problem of getting meeting context into OneNote before or during a meeting. Phase 2 closes the loop: the OneNote page becomes a complete, automatically-populated record of the meeting — what was planned, what was discussed, what was decided, and what actions were taken. Combined with the pre-day prep, it means David starts every day with structured, ready-to-use meeting pages and ends every meeting with an automatically-populated record, with no manual effort beyond running the agent.

---
*Written 22 August 2026. Not yet scheduled. Revisit after Phase 1 strategic review is complete.*
