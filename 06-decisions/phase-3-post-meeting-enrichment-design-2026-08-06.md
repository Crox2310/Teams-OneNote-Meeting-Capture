# Phase 3 design — Post-meeting enrichment (notes & actions)

## Status: DESIGN ONLY — not yet built. Captures the scoping discussion from 6 August 2026, corrected same day to accurately reflect the intended scope boundary.

## Scope statement (the ceiling, not a starting point)

**This feature is scoped to automate access to meeting information the user can already access manually, and nothing beyond that. This is not a first phase of a larger rollout — it is the full extent of what's intended.** If a user attended or was invited to a meeting, they can already open Teams and read that meeting's transcript themselves. This feature removes the manual work of doing that and writing up a summary — it does not, now or in any future phase, extend access to meetings the user wasn't part of, or to other people's data.

## The ask for today's conversation (plain English)

> "I'd like to turn on a Microsoft feature that lets my meeting notes app read the transcript of **my own** Teams meetings — the same transcript I can already open and read myself in Teams today. It needs a one-off admin approval in Entra ID to switch on. It doesn't give the app access to anyone else's meetings, doesn't need any per-user setup, and doesn't need any ongoing admin involvement after the initial approval."

That's the whole ask. Everything below is supporting detail, not additional scope to raise unless asked.

## What the feature does

Once a meeting has concluded, the user can call on an agent to review that meeting and post a summary — key points and candidate action items, derived from the transcript — back to the same OneNote page already created for that meeting. This keeps the meeting's notes and a summary of what was discussed in one place, without the user needing to manually open the transcript and write it up themselves.

## Why this stays within "only what I can already access manually"

- **Delegated permission only** (the app acts *as the signed-in user*, "me" scope) — not application permission (app acting with no signed-in user, potentially reading anyone's meetings).
- **No tenant-wide access policy, ever, for this feature.** The application-permission route would additionally require a tenant admin to run `New-CsApplicationAccessPolicy` / `Grant-CsApplicationAccessPolicy` PowerShell, per user, to authorise the app to read on someone else's behalf. **This is not part of the design, not something being asked for now, and not planned for later.**
- **Access control isn't a rule we build — it's structural.** Because the call is delegated and scoped to `/me/...`, it is not possible to retrieve a transcript for a meeting the signed-in user wasn't part of; Graph itself won't return it. There is no cross-user lookup, attendee list check, or organizer check anywhere in this design, because there's no scenario in which one identity fetches on another's behalf.
- **One permission only.** `OnlineMeetingTranscript.Read.All`, delegated — for the user's own transcript. No recording access, no other Graph permissions.

## What's being requested from IT — the single technical detail

- **Permission:** `OnlineMeetingTranscript.Read.All`, delegated.
- **Why admin consent is needed at all, even though it's "just my own data":** this specific Graph permission is flagged by Microsoft as requiring admin consent regardless of delegated vs application type — it isn't optional or something a per-app admin can skip, and it isn't a signal that the ask is broader than it looks. It's a one-time approval in Entra ID; it does not need repeating per user or per meeting.
- **Secondary, non-technical check (can be answered informally, no policy change implied):** confirm cloud recording + transcription is switched on in the Teams meeting policy assigned to David's account — without this, there's nothing for the API to fetch, independent of any permission being granted.

## Build scope once the ask is granted

**Trigger:** on-demand only (new Copilot Studio topic, e.g. "Review meeting notes", reusing the existing meeting-selection UX — P/N navigation, candidate list).

**Output:** AI-generated key points + candidate action items, derived from the user's own meeting transcript, appended to the existing OneNote page via the same PATCH/update-page-content mechanism Flow B already uses.

**Build steps (high level, not yet actioned):**
1. New Copilot Studio topic (or extension of an existing one) to select/confirm the meeting to review.
2. New child flow, delegated Graph connection (as the signed-in user) — `GET /me/onlineMeetings/{meeting-id}/transcripts`, fetch transcript content.
3. AI Builder / Copilot Studio generative action — summarise transcript into key points and candidate action items.
4. Append the output as a new HTML fragment to the existing OneNote page (reuse the "Update page content" pattern already built for the existing-page branch in Flow B).
5. Test against 3–5 real meetings of varying length/type — the main open question is whether the summarisation quality is actually good enough to be worth showing users.

## Explicitly out of scope — not planned, not deferred, not a future phase

- Reading or summarising any meeting the user wasn't invited to or didn't attend.
- Any cross-user or on-behalf-of access.
- Application permissions or tenant-wide/`CsApplicationAccessPolicy` access.
- Automatic triggering without the user asking (no scheduled polling, no auto-post on meeting end) — kept as an explicit user-initiated action, in keeping with "the same thing I could do manually."
- Recording retrieval — not needed for the summary use case and not part of this design.

## Status

**Design captured, nothing built yet, scope fixed at self-access-only.** Next step: get the single Entra ID admin consent above; only then start build work.
