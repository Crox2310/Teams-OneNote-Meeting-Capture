Handover — 6 August 2026 — Future build item: OneNote page link in Teams message is too long / wrong field
===

## Status: FUTURE BUILD — not yet actioned, no changes made to the flow or topic

## Symptom

When a meeting is captured, the success message in Teams (`C12_Success` in the Copilot Studio topic) includes a link to the OneNote page. The link renders as a very long, multi-line raw URL in Teams rather than a short, meaningful clickable link.

## Root cause — confirmed via Peek Code trace, session 6 Aug 2026

**Layer 1 — Topic message composition (easy fix, cosmetic only):**

`C12_Success` (in `04-agent-topic-flow-map/meeting-capture-topic-v1.0.1.yaml` / `12-phase-2-validation/topic-export-2026-07-31.yaml`) inserts `{Topic.OutCreatedPageLink}` directly into plain message text:

```
Great news! Your meeting notes for {Topic.MeetingTitle} have been saved to OneNote. Here's your page link: {Topic.OutCreatedPageLink}
```

Fix: use Markdown link syntax instead —
```
Great news! Your meeting notes for {Topic.MeetingTitle} have been saved to OneNote. [Open your notes]({Topic.OutCreatedPageLink})
```
This alone would shorten the *display* in Teams, but does NOT fix Layer 2 below — it would just wrap a wrong/oversized URL in a short link label, which is misleading if the underlying URL is not a genuine web-openable link.

**Layer 2 — Wrong field being used as the link value (the real bug):**

Traced via Peek Code, recurring/"existing page" branch, Flow B (`ed112c88-b94b-f111-bec6-002248a38052`):

- `Set varOutputPageLink Existing` → `value: @variables('varFinalExistingPageSelfUrl')`
- `varFinalExistingPageSelfUrl` is set by `SetVariable` → `value: @outputs('Compose_ExistingPageSelfUrl')`
- `Compose_ExistingPageSelfUrl`:
  ```
  @if(
    greater(length(body('Filter_Existing_Mapping')), 0),
    first(body('Filter_Existing_Mapping'))?['PageSelfUrl'],
    ''
  )
  ```

So on the "page already exists" (recurring, subsequent capture) path, the link given to the user is `PageSelfUrl` — a column read back from the `RecurringMeetingSectionMap` SharePoint mapping list. `PageSelfUrl` is a Graph API **self-reference URL** (e.g. `https://graph.microsoft.com/v1.0/.../pages/{id}/$value`), not a browser-openable web link. This explains both symptoms: it's needlessly long, and it may not even function correctly as a clickable link for the user (API endpoint, not `oneNoteWebUrl`).

**Confirmed: this is not a one-line fix.** The `RecurringMeetingSectionMap` list only appears to store `PageSelfUrl` (plus `SectionSelfUrl`, `SectionPagesUrl`, `SeriesMasterId`, `MeetingId`, etc. — see memory/prior handovers) for the mapping row. As far as traced so far, there is no confirmed column storing the proper OneNote web link (`links.oneNoteWebUrl.href` from the Graph API page-creation response).

## Fix plan (not yet started)

1. **Confirm** whether `RecurringMeetingSectionMap` already has a web-link column, by checking the SharePoint list columns directly or Peek Coding the "Send an HTTP request to SharePoint" action(s) that write the mapping row (both recurring and one-off branches — there are multiple write points, see OF09a/OF09b family and the recurring equivalent).
2. If no such column exists: **add a new SharePoint column**, e.g. `PageWebUrl`, to `RecurringMeetingSectionMap`.
3. At every point in Flow B where a page is **created** (both recurring and one-off), capture `links.oneNoteWebUrl.href` from the Create Page response and write it into the new `PageWebUrl` column alongside the existing `PageSelfUrl` write. Likely touches the same actions/branches as: `Set varOutputPageLink Created`, `Set varOutputPageLink Created OneOff`, `Set varOutputPageLink Created OneOff Gate`, and their corresponding SharePoint HTTP write actions.
4. Update `Compose_ExistingPageSelfUrl` (recurring "existing page" branch) — and check for an equivalent Compose action on the one-off "existing page" branch — to read the new `PageWebUrl` field instead of `PageSelfUrl`, for the *link given to the user*. `PageSelfUrl` itself should very likely be left alone/untouched for any internal logic that genuinely needs the API self-reference (e.g. update-page calls) — this is purely about what gets shown to the user.
5. Once the correct field is flowing through end-to-end (test all three scenarios: recurring, one-off new, one-off recapture — noting recapture/Bug 5 is still separately broken per the 2 August handovers), apply the Layer 1 Markdown fix to `C12_Success` in the topic.
6. Re-test live in Teams to confirm both a short display AND a genuinely working link.

## Priority / sequencing note

This is independent of, but should probably be picked up **after**, the currently-open items from the 2 August session 6 handover (Bug 5 recapture-path fix, Microsoft support ticket for the corruption pattern). Not urgent/breaking — the link currently works, it's just ugly and using the wrong field type. Low risk to defer.

## Status

**Diagnosed, not fixed. No flow or topic changes made in this session — investigation only.** Next session on this topic should start by confirming the SharePoint list's current columns (step 1 above) before making any changes.
