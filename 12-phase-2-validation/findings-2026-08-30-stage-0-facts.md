# Stage 0 findings — four factual checks

**Date:** 30 August 2026
**Session type:** Stage 0 of the 29 August backlog (`design-2026-08-29-target-state-and-backlog.md`) — four factual checks, no changes to any flow, Topic, or list (one exception: see S0.4 note on indexing, which was already in place and required no change).
**All testing done in** `PA - Scratch Diagnostics`, against the `AgentTest` notebook and production's `RecurringMeetingSectionMap` list (read-only for the latter).

---

## S0.1 — Does `<title>` in posted HTML set the page title at creation?

**Answer: FAIL.**

Tested via the native OneNote connector's `Create page in a section` action (`shared_onenote`, `CreatePageInSection`). Four valid test runs (two earlier attempts were invalidated by editor-escaping artifacts — see Note below), all using genuinely-valid HTML content, all returned the same result: the API response's `title` field came back empty (`""`), and the page appeared in OneNote's section list as "Untitled Page."

**Root cause identified:** the flow designer's Page Content parameter field strips `<html>`, `<head>`, and `<body>` wrapper tags down to fragment-only content at **Publish** time — before the action ever executes. Confirmed directly by comparing the action's Code view before and after Publish (identical content survived Save draft, then lost its wrapper tags immediately after Publish). This means a full HTML document structure can never reach this connector action via the designer UI, regardless of what is typed into Code view beforehand — so the fragment-only result is not a partial answer, it is the only obtainable answer through this action.

**Downstream impact:** routes Stage 5 to **S5.2** — the existing title-fix chain (`Set_PageTitle_Recurring`, the five-second delay, the post-create lookup) stays in place; latency work there means moving the Response action earlier, not removing the chain.

**Related item to verify separately (not confirmed this session):** if Flow B's `Create_OneNote_Page` uses this same connector action type (rather than, say, an HTTP action constructing the request some other way), then this publish-time stripping applies to production too, regardless of how carefully `text_3` builds the HTML. Worth checking before Stage 5 build work starts.

*Note on invalid runs:* the first two test attempts produced no usable signal — one was entered into the rich-text view rather than Code view and got HTML-escaped on save (the page content rendered as literal `<html>` text, not parsed markup); the second had a stray JSON key name copied into the value itself. Both were discarded rather than counted as evidence.

---

## S0.2 — Does the Teams connector's `Send a Microsoft Graph HTTP request` action reach OneNote endpoints?

**Answer: FAIL.**

Tested with a live `GET` request to `https://graph.microsoft.com/v1.0/me/onenote/pages/{pageId}` against a real, existing page ID (from an S0.1 test run). The flow run failed with an explicit connector-level rejection:

> `URI path is not a valid Graph endpoint, path is neither absolute nor relative or resource/object is not supported for this connector. Invalid resource, Allowed values: teams,me,users. Invalid Object, Allowed values: channels,chats,installedApps,messages,pinnedMessages,onlineMeetings.`

This directly confirms the allow-list documented on 28 August (`handover-2026-08-28-teams-chat-power-automate-confirmed.md`). The request was rejected by Power Automate itself before it reached Microsoft Graph at all — `onenote` is not, and apparently cannot be, a valid object segment for this connector, regardless of resource (`me`/`teams`/`users`).

**Downstream impact:** closes off the PATCH-with-`target`/`data-id` named-region insertion approach for Flow C entirely. **No shape change to Stage 5 or Stage 7** — both proceed on the existing plan (raw HTML fed to the model, deterministic `indexOf` fallback), since the DLP-approved path that would have unlocked structured OneNote writes via Teams is not available.

---

## S0.3 — Is the environment solution-aware, and does Copilot Studio's flow-invocation mechanism work against solution-aware flows?

**Answer: SPLIT — environment half CONFIRMED PASS; invocation-compatibility half UNVERIFIED.**

**Environment solution-awareness — CONFIRMED PASS.** In the `Agents - Personal Productivity` environment (ID `76f9c3bd-16c5-e540-8bb4-7171f4745b45`, the same one Flow A lives in), Power Automate's Solutions → "New solution" dialog opened as a fully functional form (Display name, Name, Publisher, Version, More options) — not blocked, no Dataverse-missing message. Existing solutions (several default/system solutions) were already visible in the list behind the dialog. No solution was actually created — dialog was closed without saving once the form's availability confirmed the answer.

**Flow-invocation compatibility — UNVERIFIED.** Two Microsoft Learn search passes ("InvokeFlowAction," then "call a flow") did not surface documentation addressing solution-awareness directly. The closest match found was "Call an agent flow from an agent" (Microsoft Copilot Studio docs), which describes a related but distinct current feature — "agent flows" added via Tools → Add a tool → Flow, requiring a `When an agent calls the flow` trigger and a `Respond to the agent` action. That page does not mention solutions or Dataverse anywhere. The term `InvokeFlowAction` itself did not appear in either search pass's results, suggesting it may be legacy/internal terminology rather than current UI language.

**Downstream impact:** genuinely best answered by live testing at Stage 7 when an actual solution-aware child flow exists to invoke against — the specific invocation mechanism Flow C will use (classic Topic-invoked child flow vs. the newer "agent flow" tool pattern) isn't decided either. Not blocking Stage 0 closure.

---

## S0.4 — Why did `Get_items` return `[]` on 21 August against a populated list, and is `SeriesMasterId` an indexed column that will accept an OData `$filter`?

**Answer: PARTIALLY RESOLVED.** Three of four candidate causes ruled out; root cause of the original 21 August empty result remains unconfirmed but is no longer a standing risk.

- **Content approval** — confirmed OFF ("Require content approval for submitted items?" → No). This was the leading hypothesis carried forward from the 21 August session (`session-2026-08-21-fb04-build-and-getitems-mystery.md`) — ruled out.
- **Competing filtered view** — ruled out. Only one view exists on the list ("All Items"), and it is simultaneously the Default View, Mobile View, and Default Mobile View.
- **Indexing** — ruled out as a cause. `SeriesMasterId` was already indexed (1 of maximum 20 indices on the list). Live test confirms it accepts an OData `$filter` cleanly — a `Get items` call with `$filter=SeriesMasterId eq '<value>'` returned a `200 OK` and the correct single row, no "column not indexed" error. **No fix needed for Stage 1's planned `$filter` change; it is safe to implement as designed.**
- **The original 21 August empty result** — unexplained, but no longer a standing risk: a live retest today, with the exact same mechanism, succeeds cleanly. The leading candidate explanation: the row tested against (`ID 296`, "Mapping", SC&L FLT Stand-up) has `Created: 2026-08-23T18:03:24Z` — *two days after* the reported 21 August failure. If the original failing run happened before this row was created, that alone fully explains a clean empty result with no error. This could not be confirmed this session because the exact timestamp of the original failing run wasn't available. Flagged as the leading explanation, not a confirmed one.

---

## Summary for Stage 5 / Stage 7 planning

| Check | Result | Shapes |
|---|---|---|
| S0.1 — `<title>` sets page title | FAIL (structural, not just empirical) | Stage 5 → S5.2. Title-fix chain stays. |
| S0.2 — Teams Graph HTTP → OneNote | FAIL (connector-level allow-list) | No shape change; Flow C proceeds on existing plan. |
| S0.3 — Solution-aware + invocation | PASS / UNVERIFIED split | Stage 7 invocation mechanism still to be live-tested. |
| S0.4 — `Get_items` empty result + indexing | Indexing confirmed safe; root cause of original failure unconfirmed | Stage 1's `$filter` change can proceed as designed. |

Two items worth carrying into the next session as open threads rather than closed:

1. Whether Flow B's `Create_OneNote_Page` uses the same connector action as tested in S0.1 (if so, the publish-time HTML-stripping applies to production too).
2. The exact timestamp of the original 21 August `Get_items` failure, to confirm or rule out the row-didn't-exist-yet explanation.
