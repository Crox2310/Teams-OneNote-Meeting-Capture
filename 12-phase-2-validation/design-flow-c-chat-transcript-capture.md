# Flow C Design: Teams Meeting Chat (and Future Transcript) Capture

**Date:** 29 August 2026
**Author:** David Croxson, Senior Head of Product, Sainsbury's
**Project:** Teams–OneNote Meeting Capture (GitHub: `Crox2310/Teams-OneNote-Meeting-Capture`)
**Status:** Design agreed, not yet built. This document exists so the build can start from a settled design rather than being figured out mid-session.
**Builds on:** `handover-2026-08-28-teams-chat-power-automate-confirmed.md` and `handover-2026-08-28-recurring-chat-scoping.md` — both confirm the Graph chain (event → joinUrl → threadId → messages) works live inside Power Automate, including against a recurring meeting.

---

## 1. Why this is a new flow, not an addition to Flow A/B

Flow A and Flow B are the validated, protected production baseline — 12+ documented corruption incidents, a fully root-caused BUG-01, and a hard-won stability record as of 23 August. Adding new, still-experimental logic (chat extraction, AI summarisation) directly into that baseline risks destabilising something that currently works correctly.

**Decision: a new flow ("Flow C") built separately, additive only.** Same principle already established for this project — diagnose, document, amend deliberately, never patch ad hoc onto working production logic.

## 2. Trigger mechanism

Flow C is triggered by a **new Topic in the same Copilot Studio agent** David already uses for meeting capture — e.g. "capture chat for [meeting]." This is semi-automatic and on-demand (David asks for it after a meeting ends), not automatic and not live/real-time — consistent with the scope agreed earlier in this design conversation. No new trigger type needed; this follows the same Topic → Flow relationship Flow A/B already use.

**Explicitly out of scope for this build:** live/real-time capture, retroactive capture of already-completed meetings not yet re-run, and transcript capture (blocked — see Section 6).

## 3. How Flow C finds the right OneNote page

**Decision: look up the meeting via the existing SharePoint mapping table (`RecurringMeetingSectionMap`)**, using `SeriesMasterId` + `OccurrenceDate` (recurring) or `MeetingId` (one-off) to retrieve `PageWebUrl`/`PageSelfUrl` — the same data Flow B already writes and maintains.

Rejected alternative: re-resolving the page fresh via calendar+subject match (the approach used for today's live testing). This was ruled out because Flow C only makes sense for a meeting that has *already* been captured by Flow A/B — you're adding chat notes to an existing page, not creating a new one — so re-doing calendar resolution work Flow B has already done is redundant.

## 4. The extraction and summarisation chain

Confirmed-working chain (from the two prior handover docs), reused as-is:

1. Resolve `joinUrl` from the calendar event's invite HTML (URL-pattern match on `https://teams.microsoft.com/l/meetup-join/`, not a template-specific anchor id — see `handover-2026-08-28-recurring-chat-scoping.md` Section 1 for why).
2. `joinUrl` → "Get an online meeting" (Teams connector) → `chatInfo.threadId`.
3. `threadId` → "Send a Microsoft Graph HTTP request" (Teams connector, `me/chats/{threadId}/messages`) → raw messages.
4. Filter to today's occurrence only (`greaterOrEquals(ticks(...))` against the calendar event's `startWithTimeZone`, no upper bound) — mitigates but does not fully solve the recurring-chat-thread-continuity risk; see the open pagination gap noted in `handover-2026-08-28-recurring-chat-scoping.md` Section 3.
5. Extract to `{speaker, timestamp, text}`, skipping messages that are purely an image with no accompanying text (a message with any real text alongside images is kept in full).

**New design decision (this document): the AI step does two jobs in one pass, not two separate steps.**
- **Judge relevance** — does this conversation contain anything worth recording (decisions, actions, risks, follow-ups, substantive discussion)?
- **If yes:** produce structured notes (grouped under something like Key Discussion / Actions / Open Questions), in the voice of notes rather than a transcript recap.
- **If no:** the flow appends a fixed marker instead — **"No additional discussion captured in chat for this session."**

Rejected alternative for the "nothing to add" case: skipping the append entirely. Rejected because every capture attempt should leave a visible trace on the page (success or explicitly "nothing to add") — a silent no-op looks identical to the feature having failed to run.

**Decision: raw HTML message bodies are fed directly to the AI summarisation step**, rather than manually stripped of HTML tags first. Rejected the manual-strip approach (regex/`replace()` chains) because the real message data captured during testing contains several different, inconsistently-shaped tags — `<at>` mentions, `<attachment>` reference blocks, and Facilitator's `<cite id="N">[N]</cite>` footnotes — that a hand-built stripper would need to handle individually and would likely miss edge cases on. An LLM parses HTML natively and can be instructed to ignore markup, removing an entire fragile step from the build.

**Confirmed out of scope for this build:** embedding actual images from chat into the OneNote page. Technically possible (binary content fetch + OneNote's multipart image-embed API, both untested today) but scoped as a clearly separate future feature, not part of this build.

## 5. OneNote page layout — new design decision, changes both Flow B and Flow C

**The problem:** Flow B's existing update mechanism (`Compose_UpdateHtmlFragment`) always prepends new content to the very top of the page, above everything already there — this works correctly today because Flow B is the only writer. Once Flow C also writes to the same page (and potentially a future transcript-summary writer too), blind top-of-page prepend means every capture piles on top of the last one in raw insertion order, with no separation between invite content, chat summaries, transcript summaries, and the user's own typed notes — and a re-run of chat capture for the same meeting would duplicate rather than replace its own prior summary.

**Decision: the OneNote page gets four named, fixed sections, each with its own anchor:**

1. **Meeting Details** — the existing calendar-invite content, written once at page creation, never touched again by any automation.
2. **User Notes** — a clearly labelled space that is explicitly the user's own; no automation writes into it, ever.
3. **Chat Summary** — Flow C writes here specifically, **replacing** its own prior content on re-capture rather than stacking a new entry above the old one.
4. **Transcript Summary** — same idea, reserved space, unused until the tenant-level transcript API block (Section 6) is lifted.

**This requires two changes, not just a new Flow C build:**
- **Flow B's page-creation template** (`Compose_UpdateHtmlFragment` and the page-creation HTML it's based on) needs these four section headers baked in from the start, as a one-time change to the production baseline. This is the one place this design touches Flow A/B directly — everything else about Flow C stays additive.
- **Flow C's append logic** needs to find and write **under a specific heading** (an HTML anchor/id search-and-insert), not the simple top-of-page prepend Flow B currently uses. This is genuinely new logic — nothing built or tested so far in this project does targeted mid-page insertion; every append tested to date has been a blind top-of-page prepend.

**Not yet decided, left for the build session:** the exact HTML/anchor mechanism for finding and replacing content under a specific heading (e.g. matching on a heading's `id` attribute, then replacing everything up to the next `<h2>`), and whether existing already-created pages (created before this template change) need any migration, or simply won't have the new sections until next updated.

## 6. Transcript status (confirmed blocker, unchanged since 28 August)

**Microsoft Graph API access to Teams meeting transcripts is disabled for this tenant** — confirmed via a direct, unambiguous error (`"Graph API access to transcripts is disabled for this tenant"`), not a permissions gap. Consent for `OnlineMeetingTranscript.Read.All` was checked directly in Graph Explorer and confirmed absent, but the error wording indicates this is a tenant-level admin/compliance setting, not something resolvable by requesting a different scope. **This is a confirmed hard stop, not an open task** — no further attempts should be made without first getting Sainsbury's Teams/Entra admin to check whether this can be enabled.

**Design intent stated by David: build for chat now, with the explicit assumption that the same downstream pipeline (extraction → judge-and-summarise → append into the Transcript Summary section) will apply once/if transcript access is unblocked.** The four-section page layout above already reserves space for this, so no further page-layout change should be needed when that day comes — only a new Section-1-equivalent extraction step feeding the same summarisation logic.

---

## 7. Summary of what's settled vs. still open

**Settled by this document:**
- New, separate Flow C — confirmed rationale (protect the baseline).
- Trigger: new Copilot Studio Topic, on-demand only.
- Page lookup: via the SharePoint mapping table, not fresh calendar resolution.
- AI step: combined judge-and-summarise, single pass, raw HTML input.
- "Nothing to add" case: explicit marker, not a silent skip.
- Images: skipped entirely for this build; real embedding deferred as a future feature.
- Page layout: four fixed, named sections (Meeting Details / User Notes / Chat Summary / Transcript Summary).
- Transcript: confirmed hard-blocked at the tenant level; chat-only for now, transcript slot reserved for later.

**Still open, for the build session:**
- Exact anchor/replace mechanism for Flow C's mid-page insertion.
- Whether/how already-existing pages get migrated to the four-section template.
- The exact AI prompt wording for the judge-and-summarise step.
- The still-outstanding pagination gap in recurring-chat scoping (Section 3, `handover-2026-08-28-recurring-chat-scoping.md`) — not resolved by this document, remains a genuine open risk for chatty recurring meetings.
