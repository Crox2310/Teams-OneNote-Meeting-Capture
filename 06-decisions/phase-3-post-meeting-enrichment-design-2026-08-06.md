# Phase 3 design — Post-meeting enrichment (notes & actions pilot)

## Status: DESIGN ONLY — not yet built. Captures the scoping discussion from 6 August 2026, tightened same day to keep the admin ask minimal and low-risk.

## The one ask for today's conversation (plain English)

> "I'd like to turn on a Microsoft feature that lets my meeting notes app read the transcript of **my own** Teams meetings — the same transcript I can already open and read myself in Teams today. It needs a one-off admin approval in Entra ID to switch on. It doesn't give the app access to anyone else's meetings, doesn't need any per-user setup, and doesn't need any ongoing admin involvement after the initial approval."

That's the whole ask. Everything below is supporting detail, not additional scope to raise unless asked.

## The ambition (long-term — not part of today's ask)

Longer-term, the goal is for a user to call on an agent to review a concluded meeting and post output material back to the OneNote page already created — key points, action items, eventually a recording link — so all artefacts related to a meeting live in one place. This is background context only; **today's ask is scoped to a much smaller first step**, not this full picture.

## Deliberately reduced scope: "only what I can already access manually"

The guiding constraint for this phase, and for the ask itself, is: **the feature must never access anything the user couldn't already open and read themselves.** Concretely, that means:

- **Delegated permission only** (the app acts *as the signed-in user*, "me" scope) — not application permission (app acting with no signed-in user, potentially reading anyone's meetings).
- **No tenant-wide access policy.** The application-permission route would additionally require a tenant admin to run `New-CsApplicationAccessPolicy` / `Grant-CsApplicationAccessPolicy` PowerShell, per user, to authorise the app to read on someone else's behalf. **This pilot does not need that, and is not asking for it.**
- **Access control isn't a rule we build — it's structural.** Because the call is delegated and scoped to `/me/...`, it is not possible to retrieve a transcript for a meeting the signed-in user wasn't part of; Graph itself won't return it. There's nothing extra to build or explain here — the permission model already enforces "only my own meetings."
- **One permission, nothing else, for the pilot.** Recording access (`OnlineMeetingRecording.Read.All`) is a separate permission and is deliberately left out of the pilot ask — it isn't needed to prove the core idea (AI-summarised key points and action items from a transcript).

## What's being requested from IT — the single technical detail

- **Permission:** `OnlineMeetingTranscript.Read.All`, delegated.
- **Why admin consent is needed at all, even though it's "just my own data":** this specific Graph permission is flagged by Microsoft as requiring admin consent regardless of delegated vs application type — it isn't optional or something a per-app admin can skip, and it isn't a signal that the ask is broader than it looks. It's a one-time approval in Entra ID; it does not need repeating per user or per meeting.
- **Secondary, non-technical check (can be answered informally, no policy change implied):** confirm cloud recording + transcription is switched on in the Teams meeting policy assigned to David's account — without this, there's nothing for the API to fetch, independent of any permission being granted.

## Pilot scope once the ask is granted (unchanged from design discussion, kept narrow)

**Trigger:** on-demand only (new Copilot Studio topic, e.g. "Review meeting notes", reusing the existing meeting-selection UX — P/N navigation, candidate list). No auto-post-on-meeting-end, no scheduled polling, for the pilot.

**MVP output:** AI-generated key points + candidate action items only, appended to the existing OneNote page via the same PATCH/update-page-content mechanism Flow B already uses. No recording link in the pilot.

**Build steps (high level, not yet actioned):**
1. New Copilot Studio topic (or extension of an existing one) to select/confirm the meeting to review.
2. New child flow, delegated Graph connection (as the signed-in user) — `GET /me/onlineMeetings/{meeting-id}/transcripts`, fetch transcript content.
3. AI Builder / Copilot Studio generative action — summarise transcript into key points and candidate action items.
4. Append the output as a new HTML fragment to the existing OneNote page (reuse the "Update page content" pattern already built for the existing-page branch in Flow B).
5. Test against 3–5 real meetings of varying length/type — the main open question is whether the summarisation quality is actually good enough to be worth showing users, before investing in anything else around it.

## Deliberately deferred — not part of this ask or this pilot

- Auto-post trigger (scheduled polling or Graph change-notification subscription).
- Recording link retrieval.
- Cross-user / attendee-list access control — only relevant if a future phase needs to fetch on someone else's behalf, which is explicitly not what's being asked for now.
- Application-permission + `CsApplicationAccessPolicy` route — only if the feature ever needs to scale beyond "each user runs this for themselves." Not requested, not needed today.

## Status

**Design captured, nothing built yet, ask deliberately minimised for today's conversation.** Next step: get the single Entra ID admin consent above; only then start build work.
