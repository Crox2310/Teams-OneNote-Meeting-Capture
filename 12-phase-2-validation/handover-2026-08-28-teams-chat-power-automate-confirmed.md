# Confirmed: Teams Chat Extraction Chain Proven Live Inside Power Automate

**Date:** 2026-08-28
**Author:** David Croxson, Senior Head of Product, Sainsbury's
**Project:** Teams–OneNote Meeting Capture (GitHub: `Crox2310/Teams-OneNote-Meeting-Capture`)
**Supersedes/extends:** `handover-2026-08-28-teams-chat-extraction-graph-confirmed.md` — that document proved the three-step Graph chain via curl and Graph Explorer, then stated an untested assumption: "Power Automate's HTTP + Microsoft Entra ID action builds requests differently, so it's not expected to hit the same issue [as Graph Explorer's UI]." **That assumption has now been tested directly and found to be moot** — not because it was wrong, but because that connector is blocked entirely in this tenant, and a better path was found instead.

---

## 1. Headline finding: DLP blocks raw HTTP-to-Graph in this tenant

Attempting to create a connection for **"HTTP with Microsoft Entra ID"** (the Premium connector originally planned for this work) inside Copilot Studio / Power Automate failed immediately with:

> *"Connection creation/edit of 'HTTP With Microsoft Entra ID' has been blocked by Data Loss Prevention (DLP) policy 'Copilot Studio Default Policy'."*

This is a tenant-wide governance rule, not a permissions or configuration issue on this specific flow. It cannot be worked around from within Designer — it would require a policy change from whoever administers the Power Platform environment. **No further attempts should be made to use this connector for this or related work without that policy change.**

## 2. The working alternative: Microsoft Teams connector's native actions

Two actions from the (non-blocked) **Microsoft Teams** connector cover the same ground the blocked HTTP connector would have, with one caveat (see Section 3):

- **"Get an online meeting"** — accepts a `joinWebUrl` directly via its "Lookup by" parameter set to "Join web URL." Confirmed live: returns the full online meeting object including `chatInfo.threadId`.
- **"Send a Microsoft Graph HTTP request"** — despite the name, this is **not** a free-form HTTP caller. It's a structured OpenAPI connection under the hood (`shared_teams` / operation `HttpRequest`) that validates the URL path against an explicit allow-list:
  - Resource must be one of: `teams`, `me`, `users`
  - Object must be one of: `channels`, `chats`, `installedApps`, `messages`, `pinnedMessages`, `onlineMeetings`
  - Confirmed working form for pulling chat messages: `https://graph.microsoft.com/v1.0/me/chats/{threadId}/messages` — note the `me/` prefix is required; `chats/{threadId}/messages` alone fails, because `chats` alone is read as the resource segment (invalid) rather than the object segment.

**Both actions are confirmed live and working**, tested against the same real meeting used in the original handover doc ("Discussion on graph access," 26 Aug 2026).

## 3. The caveat: no direct calendar-event lookup via this connector

"Send a Microsoft Graph HTTP request" does **not** support `events` as an object (only the six listed above), so it cannot replace Step 1 of the original three-step chain (resolving a calendar event to its `joinUrl`). Two options were considered:

- Use a different, non-blocked connector/action for Step 1 specifically — not yet identified.
- **Extract the classic-format join URL directly from data already being pulled** — the existing "Get calendar view of events (V3)" action's `body` field (the event's HTML invite text) already contains the join link, in the exact classic `meetup-join` format needed. This avoids needing Step 1's HTTP call at all.

**The second option was built and confirmed working.** It requires pulling the *second* Teams link in the body HTML (anchor id `meet_invite_block.action.join_link_compatibility`), not the first "Join" link (which is the newer short-form `teams.microsoft.com/meet/...` format and does not match what "Get an online meeting" expects). This was built as a chain of Compose actions doing string extraction (`indexOf`/`substring`/`lastIndexOf`), each producing an intermediate value for the next, tested step by step.

## 4. Full chain, confirmed live end-to-end in a real flow run (not curl, not Graph Explorer)

1. **Get calendar view of events (V3)** (existing Office 365 Outlook action) → raw event list including `body` HTML.
2. **Filter Array** on that list, matching `subject` to the target meeting → isolates one event.
3. **Chained Compose actions** (string extraction) → pulls the classic-format join URL out of the event's `body` HTML.
4. **Get an online meeting** (Teams connector), Lookup by "Join web URL," value = the extracted join URL → returns `chatInfo.threadId`.
5. **Send a Microsoft Graph HTTP request** (Teams connector), GET `https://graph.microsoft.com/v1.0/me/chats/{threadId}/messages` → returns the chat messages.

All five steps ran successfully in a single test execution. Step 5's output matched the earlier curl-based test in the prior handover doc exactly: 9 messages, 4 system/event messages (call started/ended, chat renamed, member joined) and 5 genuine messages (including the same image-only message via `hostedContents` noted previously). This confirms the filtering finding from the earlier document (`messageType eq 'message' and from ne null`) still applies unchanged.

## 5. Lessons worth carrying into the real build (not just this test)

- **"Send a Microsoft Graph HTTP request" is not free-form** — always check its Code view / structured parameters rather than assuming a pasted URL behaves like a generic HTTP call. Any future Graph object needed via this connector must be checked against its resource/object allow-list first.
- **Filter Array's Advanced-mode condition field is already expression-typed** — do not prefix expressions with `@` in that field (unlike Compose/URI fields, which require it). Using `@` there produces "The expression is invalid" even though the expression itself is syntactically correct.
- **Renaming an action after other actions already reference it does not update those references** — always rename an action immediately after adding it, before wiring anything else to it. Retyping expressions after a late rename is how several avoidable errors were introduced during this session.
- **Cut/paste to fix a broken `runAfter` does not recompute it** — the pasted action keeps its old `runAfter` pointer. `runAfter` must be corrected directly (via Code view) rather than assumed to fix itself from a reposition.
- **DLP policy blocks should be checked early**, ideally before designing around a specific connector, given how this tenant is configured — worth a quick test connection attempt on any new Premium connector before building logic around it.

## 6. What this means for the actual feature build

The three-step design in the prior handover doc is now **fully de-risked for implementation**, with a concrete, tested action sequence rather than a theoretical one. The build described in that document's Section 7 ("What's still left to build") can proceed using the pattern confirmed here — specifically:

- Step 1 of the feature build should use the **body-HTML extraction pattern** (Section 3 above), not a direct HTTP call to `/events/{id}`.
- Steps 2 and 3 should use the **Teams connector's native actions** (Section 2 above), not "HTTP with Microsoft Entra ID," which remains DLP-blocked.
- The filter/HTML-strip/summarise/append work described in the prior handover doc's Sections 4, 5, and 7 is unaffected and still applies as written.

---

**Bottom line:** the chain is proven, live, inside Power Automate, dynamically — no hand-copied test data, every action fed by the one before it. The DLP block turned out to be a real constraint, not a false alarm, and the Teams-connector-native path found instead is arguably a cleaner design than the originally planned raw-HTTP approach.
