# Phase 3 design — Post-meeting enrichment (notes & actions pilot)

## Status: DESIGN ONLY — not yet built. Captures the scoping discussion from 6 August 2026.

## The ambition (long-term)

Once a meeting has concluded, the user should be able to call on an agent to review the meeting and post output material back to the same OneNote page Flow B already created — recording link, transcript, AI-summarised key points, action items — so all artefacts related to a meeting live in one place. Benefit: the user doesn't need to attend live and can catch up later, with everything already gathered for them.

## Feasibility reality-check (this session)

Before committing to the full ambition, we pressure-tested what's actually required via Microsoft Graph, and deliberately cut scope down to a provable pilot rather than building the whole thing speculatively.

### Graph API landscape

- **Transcripts**: `callTranscript` resource. `GET /users/{id}/onlineMeetings/{meeting-id}/transcripts` (or `/me/...` for delegated), then fetch transcript content. Only works while the meeting/transcript hasn't expired per Teams retention limits.
- **Recordings**: `callRecording` resource, similar list/fetch shape. Separate permission from transcripts.
- **No dedicated "meeting notes" or "action items" API** — there isn't a direct Graph endpoint for meeting notes. Action items have to be derived (via AI summarisation of the transcript), not fetched ready-made.
- **Permissions**:
  - `OnlineMeetingTranscript.Read.All` (and `.Read.Chat` if application-permission route) — **requires admin consent even for delegated scenarios**, because the permission's `AdminConsentRequired` flag is set to Yes regardless of delegated vs application type. This is a fixed, one-time cost no matter how small the pilot is.
  - **Application permissions** (app acting with no signed-in user, reading *anyone's* meetings) additionally require a tenant admin to run `New-CsApplicationAccessPolicy` / `Grant-CsApplicationAccessPolicy` PowerShell per user whose meetings the app should access. This is the expensive, rollout-scale dependency.
  - **Delegated permissions** (app acting as the signed-in user, "me" scope) avoid the application-access-policy step entirely — the flow only ever sees what the signed-in user could already see themselves.
- Also worth confirming with IT: whether cloud recording + transcription is even switched on in the relevant Teams meeting policy — if off, there's nothing to fetch regardless of API access granted.

## Key design decision: delegated ("me") scope only, day one

**Access control is not a rule we build — it's inherent to delegated permissions.** If Flow C calls Graph as the signed-in user (`/me/onlineMeetings/{id}/transcripts`), it is structurally impossible to retrieve a transcript for a meeting that user wasn't part of — Graph itself won't return it. This means:

- No attendee/organizer-list check needs to be built for the pilot.
- The feature only ever automates what the user could already do manually (open Teams, find the meeting, read the transcript, copy a summary) — it doesn't expose any new access. This is a useful, precise answer if IT or management ask what the feature exposes: **nothing beyond what the user could already see themselves.**
- A tenant-wide, cross-user access-control check (attendee list comparison, organizer lookup) is deferred to a later phase, only needed if/when this moves beyond "each user runs this against their own meetings" toward any scenario where one identity might fetch on behalf of another.

## Pilot scope (deliberately narrow)

**Trigger:** on-demand only for the pilot (e.g. new Copilot Studio topic "Review meeting notes", reusing the existing meeting-selection UX pattern — P/N navigation, candidate list). Auto-post-on-meeting-end (scheduled polling flow, "already processed" tracking) is deferred — it's meaningfully more infrastructure (polling flow, dedup tracking column on the mapping list) and isn't needed to prove the core value.

**MVP output:** AI-generated key points + candidate action items, appended to the existing OneNote page via the same PATCH/update-page-content mechanism Flow B already uses. Recording link is deferred — it's a separate permission (`OnlineMeetingRecording.Read.All`) and a separate URL-shape problem (recording links typically land in Stream/SharePoint, similar in spirit to the `PageSelfUrl` vs `oneNoteWebUrl` issue already found and logged elsewhere), not needed to prove whether transcript-derived summaries are actually useful.

**Build steps (high level, not yet actioned):**
1. New Copilot Studio topic (or extension of an existing one) to select/confirm the meeting to review.
2. New child flow, delegated Graph connection (as the signed-in user) — `GET /me/onlineMeetings/{meeting-id}/transcripts`, fetch transcript content.
3. AI Builder / Copilot Studio generative action — summarise transcript into key points and candidate action items. (Genuinely needs an LLM step here, unlike the Email Triage digest which was deliberately made deterministic — summarisation isn't rule-based.)
4. Append the output as a new HTML fragment to the existing OneNote page (reuse the "Update page content" pattern already built for the existing-page branch in Flow B).
5. Test against 3–5 real meetings of varying length/type — the main open question is whether the summarisation quality is actually good enough to be worth showing users, before investing in anything else around it (auto-trigger, recordings, cross-user access control).

## Deferred to later phases (not pilot blockers, but noted so they aren't lost)

- Auto-post trigger (scheduled polling or Graph change-notification subscription once transcript/recording available) — polling recommended over change notifications initially, since notifications need a publicly reachable HTTPS endpoint with validation handshake and renewal logic.
- Recording link retrieval and its own web-URL-shape validation.
- Cross-user / attendee-list access control — only needed if the feature ever needs to fetch on behalf of someone other than the requesting user.
- Application-permission + `CsApplicationAccessPolicy` route — only if the feature needs to scale beyond "each user runs this for themselves."

## Immediate action needed (outside this project, blocks all pilot work)

Ask Sainsbury's M365/Entra admin for:
1. Admin consent in Entra ID for `OnlineMeetingTranscript.Read.All` (delegated) — one-time, single click, not an ongoing burden.
2. Confirmation that cloud recording + transcription is enabled in the Teams meeting policy assigned to David's account.

## Status

**Design captured, nothing built yet.** Next session on this topic should start with confirming the two admin-dependency items above before any flow/topic build work begins.
