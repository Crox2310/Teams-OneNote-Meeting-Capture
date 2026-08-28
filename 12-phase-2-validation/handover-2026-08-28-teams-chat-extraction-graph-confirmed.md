# Handover Notes: Teams Chat Extraction for OneNote Meeting Capture (Phase 2)

**Date:** 2026-08-28
**Author:** David Croxson, Senior Head of Product, Sainsbury's
**Project:** Teams–OneNote Meeting Capture (GitHub: `Crox2310/Teams-OneNote-Meeting-Capture`)
**Purpose of this note:** Handover to a new Claude chat session so work can continue without re-deriving what's already been learned.

---

## 1. What we were trying to do

Explore a **future, on-demand feature** for the existing Meeting Capture agent (Copilot Studio + Power Automate): after a Teams meeting finishes, pull the meeting's **chat** (not the transcript), summarise it, and append that summary into the existing OneNote page already created for that meeting by the current v2 Clean Build baseline.

**Scope constraints (explicit, unchanged):**
- Semi-automatic — triggered on-demand by David asking for it, not automatic.
- Future meetings only — no retroactive work on meetings already captured in OneNote/SharePoint.
- Must not disturb the existing, protected v2 Clean Build baseline (Flow A / Flow B) — this is additive, not a rebuild.

Testing approach: verify every Graph call **live**, one step at a time, in Graph Explorer and/or curl, before writing anything into Power Automate. No assumptions about API shape.

---

## 2. Headline outcome: we have a confirmed, working path through

All three Graph API steps needed to get from "a meeting happened" to "here is its chat content" have now been **tested live against real data and confirmed working**. This is no longer a research question — it's a solved capability, ready to be built into the flow.

---

## 3. The three-step Graph chain (all confirmed working)

### Step 1 — Resolve a calendar event to its Teams joinUrl
```
GET https://graph.microsoft.com/v1.0/me/events/{eventId}?$select=isOnlineMeeting,onlineMeeting
```
- **Status:** Confirmed working.
- Returns `isOnlineMeeting: true` and a populated `onlineMeeting.joinUrl`.
- Important nuance: this is the **classic `meetup-join` format** joinUrl, not the newer short-form `teams.microsoft.com/meet/...` link that appears in invite email bodies. The classic format is what Step 2's filter expects.
- Permission used: whatever Graph Explorer's default scopes provide (at minimum `Calendars.Read`).

### Step 2 — Resolve the joinUrl to the chat's threadId
```
GET https://graph.microsoft.com/v1.0/me/onlineMeetings?$filter=JoinWebUrl eq '{joinUrl}'
```
- **Status:** Confirmed working — but only via **curl**, not via Graph Explorer's own UI.
- **What went wrong initially:** Running this in Graph Explorer's address bar repeatedly returned `400 Bad Request` / `InvalidArgument`, message: *"One of the required parameters to lookup meeting by QueryOptions is null or empty."* Explorer's own linter also flagged "Possible error found in URL" around the encoded sequences.
- **Root cause, confirmed (not just theorised):** This was **Graph Explorer's browser UI mishandling the heavily percent-encoded filter string**, not a real problem with the query, the encoding, or permissions. Proof: the identical URL, run via `curl` with a bearer token copied from Explorer's own Access Token tab, succeeded immediately with a clean `200 OK` and a full JSON body.
- **Encoding note for future reference:** A Teams `joinUrl` already contains literal percent-encoded characters (`%3a`, `%40`, `%7b`, `%22`) as part of its normal format. Encoding that whole string once (correctly) to sit inside another URL's query string turns every existing `%` into `%25` — which visually looks like "double encoding" but is actually a single, correct encoding pass of a string that happens to already contain percent signs. This was verified computationally (Python `urllib.parse.quote`) and matched byte-for-byte what was manually built — so the manual encoding was never the bug.
- **Permissions confirmed not the issue:** `OnlineMeetings.Read` (delegated) was already consented before this test — Graph Explorer's Modify Permissions tab showed it as "Unconsent" (i.e., already active/granted), no admin consent required for this scope.
- Response includes `chatInfo.threadId` and `chatInfo.messageId` — this is the key value carried into Step 3.

### Step 3 — Retrieve the chat messages
```
GET https://graph.microsoft.com/v1.0/chats/{threadId}/messages
```
(Note: the `@` in the threadId must be encoded as `%40` when placed in a URL.)
- **Status:** Confirmed working on first attempt, via curl.
- Permission used: `Chat.Read` (delegated) — already consented, "Admin consent required: No" confirmed in Graph Explorer's permissions panel.
- Tested against a real meeting ("Discussion on graph access", 26 Aug 2026, 25m46s duration) and returned 9 real message objects.

---

## 4. Key finding: raw chat payload needs filtering before it's usable

The 9 messages returned in the Step 3 test split into two clearly distinguishable categories:

**System/event messages (4 of 9)** — not real conversation content:
- `messageType: "unknownFutureValue"`, `from: null`, actual content lives in an `eventDetail` object instead of `body`.
- Examples seen: member joined chat, chat renamed, call started, call ended (with `callDuration` and `callParticipants`).

**Genuine chat messages (5 of 9)** — real conversation content:
- `messageType: "message"`, populated `from.user.displayName`, real `body.content`.
- Examples seen: plain text messages, a shared hyperlink, and one **image-only message** (content delivered via a `hostedContents` reference, no text body at all).

**Filter required before any summarisation step:**
```
messageType eq 'message' and from ne null
```
In Power Automate this is a straightforward **Filter Array** action on the `value` collection returned by Step 3. Without this filter, system noise (call start/end, joins, renames) would get fed into the summariser alongside actual conversation.

---

## 5. Two open design decisions (not yet resolved — deliberately parked)

1. **HTML-encoded body content.** All real message bodies come back HTML-encoded (e.g. `&lt;p&gt;...&lt;/p&gt;`). Will need an HTML-strip step before summarisation — similar in spirit to existing transcript-handling logic elsewhere in the project, but not yet built for this path.
2. **Image-only messages.** One of the 5 real messages in testing had no text at all — just an image via `hostedContents`. Decision not yet made between: (a) ignore/skip, (b) insert a placeholder like "[image shared]", or (c) actually fetch and embed the image via a follow-up `hostedContents/{id}/$value` call.

Neither of these blocks progress — they're implementation choices for whoever builds the next stage, not open technical risks.

---

## 6. How this fits the existing Meeting Capture architecture

This is additive to the current (protected) v2 Clean Build baseline, not a rebuild of it:

- **Flow A** (resolve meeting selection) already resolves the target event and its `CalendarEventId`. Step 1's joinUrl lookup is a natural small addition here, reusing data already being fetched.
- **Flow B** (OneNote section/page resolution and append) already has a working, tested **append-only update path** (`PAGE_UPDATED_APPEND`, using `UpdateHtmlFragment` composed and applied via the existing "Update page content — Existing Branch" action). A chat summary is just one more HTML fragment appended through that same proven mechanism — not a new page-creation or persistence pattern.
- This preserves the explicit project rule: **diagnose first, document, amend the baseline deliberately — no ad hoc patching.** Nothing about this work has touched or put at risk the current safe baseline.

---

## 7. What's still left to build (none of it technically blocked)

1. Filter step: `messageType eq 'message' and from ne null` (Filter Array action).
2. HTML-strip step on `body.content`.
3. Resolve the image-only-message design decision (see Section 5).
4. Summarisation step (likely a Compose/AI Builder or Copilot Studio prompt action) over the filtered, cleaned messages.
5. Wire the resulting summary into Flow B's existing append pattern — likely as a new optional input alongside the existing `PageHtml` field.
6. New on-demand trigger — a Copilot Studio topic/utterance (e.g. "summarise the chat for that meeting") rather than anything automatic, consistent with the on-demand-only scope constraint.
7. Design intent still to implement (not yet built): at capture time going forward, store the event's `joinUrl` (tentatively a new field `JoinWebUrl`) alongside the existing SharePoint mapping data, so a future "pull chat for this meeting" action doesn't need to re-query the calendar from scratch.

---

## 8. Recommended next step for whoever picks this up

Follow David's established build pattern (documented preference, used successfully on prior stages of this project): **incremental rebuild from a clean baseline, stage-by-stage testing, explicit checkpoints, exposing intermediate outputs via Respond to Agent.**

Suggested order:
1. Test the filter + HTML-strip logic in isolation first (standalone Compose actions in Power Automate), against the real 9-message test payload already captured — before touching Flow A or Flow B at all.
2. Once that's validated, sketch the Flow A/B amendments as a formal baseline-amendment proposal (per David's stated preference: diagnose → document → list required amendments → avoid ad hoc patching) rather than editing the live flows directly.
3. Resolve the image-handling decision (Section 5) before or during that amendment design, since it affects the shape of the filter/summarise steps.

---

## 9. Reference data from live testing (for continuity)

- Test meeting: "Discussion on graph access", 26 Aug 2026, organiser Aravindkumar A, attendee David Croxson, duration 25m46s.
- Confirmed threadId format: `19:meeting_{base64-like string}@thread.v2`.
- Confirmed permissions already active on David's account (delegated, no admin consent needed): `OnlineMeetings.Read`, `Chat.Read`, `Chat.ReadBasic`.
- Working theory that proved correct: Graph Explorer's own request-builder UI, not encoding or permissions, was the cause of the Step 2 blocker. Curl (or any client that isn't Explorer's UI, e.g. Power Automate's HTTP + Microsoft Entra ID action) is not expected to hit the same issue.

---

**Bottom line for the new chat:** The Graph API path from "meeting event" to "usable chat messages" is proven end-to-end. The remaining work is straightforward flow-building (filter, clean, summarise, append) plus two small design decisions — not further API research.
