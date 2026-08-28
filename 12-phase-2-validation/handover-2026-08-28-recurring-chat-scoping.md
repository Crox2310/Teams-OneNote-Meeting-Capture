# Recurring-Meeting Chat Scoping: Findings and Open Gap

**Date:** 2026-08-28
**Author:** David Croxson, Senior Head of Product, Sainsbury's
**Project:** Teams–OneNote Meeting Capture (GitHub: `Crox2310/Teams-OneNote-Meeting-Capture`)
**Supersedes/extends:** `handover-2026-08-28-teams-chat-power-automate-confirmed.md` — that document proved the chat-extraction chain against a one-off meeting ("Discussion on graph access"). This document extends the same chain to a **recurring** meeting ("CFSC&L Portfolio Delivery Review") and addresses the recurring-series chat-scoping risk the earlier document had not yet tested.

---

## 1. Finding: Teams invite HTML template is not consistent across meetings

The body-HTML extraction logic built and proven in the earlier handover doc (hunting for the anchor id `meet_invite_block.action.join_link_compatibility`) **failed** against "CFSC&L Portfolio Delivery Review" with a `substring` "start index out of range" error, because `indexOf()` correctly returned `-1` — that anchor id genuinely does not exist in this meeting's invite.

**Root cause:** this meeting's invite uses a different/newer Teams invite template. It has only one Teams link ("Join the meeting now"), already in the classic `l/meetup-join/` format, with **no `id` attribute at all** — unlike the original test meeting's invite, which had two links (a short-form one, then a second classic-format one carrying that specific `id`).

**Fix, more robust than the original approach:** search for the **URL pattern itself** (`https://teams.microsoft.com/l/meetup-join/`) rather than a template-specific anchor `id`. This pattern is present in both invite templates seen so far. Updated expressions:

- **SD12 — Compose BodyBeforeId** (now really "find the index of the classic join URL"):
```
indexOf(first(body('SD11_FilterEvent'))?['body'], 'https://teams.microsoft.com/l/meetup-join/')
```
- **SD13 — Compose UrlStart** simplifies to a pass-through, since `indexOf` already gives the exact start position:
```
outputs('SD12_—_Compose_BodyBeforeId')
```
- SD14 and SD15 are unchanged.

**Confirmed working** against the recurring meeting — full chain ran green end-to-end, "Get an online meeting" resolved correctly, and SD16 pulled live chat messages. This fix supersedes the original id-hunting approach for all future builds; the id-based version should not be reused.

## 2. Finding: Facilitator (Copilot-in-meeting) messages confirmed reachable via chat

A message from Teams' **Facilitator** bot was captured live in the "CFSC&L Portfolio Delivery Review" chat — a mid-meeting "About 10 minutes remain..." summary referencing portfolio items with `<cite id="N">[N]</cite>` footnote markup.

Its `from` field:
```json
"from": { "user": null, "application": { "displayName": "Facilitator", "applicationIdentityType": "bot" } }
```

This confirms the filter already specified in the earlier handover doc (`messageType eq 'message' and from ne null`) works exactly as predicted — `from` is populated (via `application`, not `user`), so Facilitator messages pass the filter and are captured. No filter change needed. **New note for the HTML-strip step (still to be built):** Facilitator messages contain `<cite id="N">[N]</cite>` citation tags, which a generic HTML stripper may not handle cleanly — worth specific attention when that step is built.

## 3. Recurring-series chat scoping — fix built, but only partially validated

**The risk (raised by David, not yet covered in prior docs):** a recurring meeting's Teams chat thread is continuous across all occurrences. Pulling "the chat" for this week's occurrence risks also pulling every previous week's conversation mixed in.

**Fix implemented:** a new Filter Array action, `SD17_FilterTodaysMessages`, added after `SD16_GetChatMessages`:
- **From:** `body('SD16_GetChatMessages')?['value']` — note: **not** `body('SD16_GetChatMessages')?['body']?['value']`. `body()` on an HTTP-request-style action already resolves to the inner response object, so an extra `?['body']` was initially included in error and returned `Null`. Worth flagging as a repeatable gotcha for any future `body()` reference on this style of action.
- **Advanced-mode condition** (no leading `@`, same rule as other Filter Array conditions in this flow):
```
greaterOrEquals(ticks(item()?['createdDateTime']), ticks(first(body('SD11_FilterEvent'))?['startWithTimeZone']))
```
- Deliberately **no upper bound** — since this is pulled on-demand shortly after a meeting ends, there's no risk of scooping up a future occurrence's messages, and setting an artificial end-time cutoff risked chopping off legitimate post-meeting wrap-up chat (confirmed present in testing — real discussion continued until 09:30, half an hour after the 09:00 scheduled end).
- `ticks()` used on both sides rather than comparing date strings directly, since `createdDateTime` and the calendar event's date fields use slightly different formatting (`Z` vs `+00:00`) that could cause unreliable string comparison.

**Confirmed:** SD17 runs without error and correctly returns all 20 messages from the test pull — but this is **not yet a positive confirmation that the filter successfully excludes old messages**, because every message in the tested page fell after today's start time; nothing in this particular dataset exercised the exclusion path.

**Remaining open gap, not yet resolved:** `SD16_GetChatMessages`'s response includes `@odata.nextLink`, meaning the chat has more than one page of messages (only the first ~20 were pulled). Older messages — very plausibly including last week's occurrence's conversation — are likely sitting beyond that link, unfetched. This means:
- The date filter (SD17) is proven correct **in principle** and will work if old messages ever land on the same page as new ones.
- But the flow as it stands **only fetches page 1**, so if a chatty meeting pushes last week's content past that boundary, it's simply never retrieved — which happens to produce the right outcome for the wrong reason (never encountering the old data, rather than correctly filtering it out).
- **Not yet tested:** what happens on a quieter recurring meeting, or one closer to its pagination boundary, where old and new messages might genuinely coexist on the same page. This is the actual stress test the current build hasn't been put through.

**Recommendation for whoever builds this properly:** don't treat "only fetches page 1" as sufficient by design. Either fetch until `@odata.nextLink` is exhausted or a message pre-dating today's start is encountered (early-exit), or explicitly document why single-page-only is an acceptable limitation for the production build. This should be resolved before treating recurring-meeting chat capture as production-ready.

## 4. Updated full chain (supersedes the version in `flow-definition-2026-08-28-scratch-diagnostics-chat-chain.md`)

Manual trigger → Get calendar view of events (V3) → SD11_FilterEvent (filter by subject) → **SD12/SD13 (updated per Section 1)** → SD14/SD15 (unchanged) → Get an online meeting → SD16_GetChatMessages → **SD17_FilterTodaysMessages (new, per Section 3)**.

---

**Bottom line:** the chain now generalises across at least two different Teams invite templates, and Facilitator/Copilot messages are confirmed reachable through the existing chat-extraction path with no special-casing needed. The recurring-series scoping fix is logically sound and partially tested, but genuine confirmation that it excludes old content — rather than never encountering it — is still outstanding and should be treated as an open item, not a closed one.
