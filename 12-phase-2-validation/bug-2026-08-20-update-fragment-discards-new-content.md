# Bug — recapture appends a generic notice, never writes the organiser's updated content (20 August 2026)

## Status
**Confirmed via live test, root cause identified in code. Not yet fixed.**

## Origin
Raised by David as field observation #2 on 20 August 2026 alongside the per-occurrence recurring-page issue (see `design-amendment-2026-08-20-per-occurrence-recurring-pages.md`). Original ask: when a one-off meeting is recaptured after the organiser has updated the meeting content, the new content should be appended, with existing human-entered notes on the page protected from being overwritten.

## What we expected vs what happens

**Expected**: existing page content preserved, new meeting content (from the fresh `PageHtml` trigger input) appended below it.

**Actual (confirmed live, 20 Aug 2026)**: existing page content is genuinely preserved — but nothing from the new capture is written anywhere. Instead, a static, generic notice is appended: *"Meeting details were refreshed by the automation. Existing human-entered notes were preserved."* The organiser's actual updated content is silently discarded.

## Test performed

1. Created throwaway one-off meeting "Test Meeting 1" (not part of a series).
2. First capture: confirmed clean create — plain page content, meeting body "Message A", no "Automated update" block. `MeetingId`: `AAMkAGY0OGU4Mzk5LWQ4NTYtNDU4MS1hY2YyLTQxOWYwZjhiMWM1ZQBGAAAAAADWkXK1vW2mQ4SwNGpyD7SzBwB8mPnOPkRmT5-MxNoNopoPAAAAAAENAAB8mPnOPkRmT5-MxNoNopoPAAnhQdQ8AAA=`. `SeriesMasterId` (`text_2`) empty, confirming one-off path.
3. Edited the meeting body in Outlook: "Message A" → "Message BBBB". Title left unchanged.
4. Second capture: raw trigger body confirmed `MeetingId` identical to the first capture, and `text_3` (`PageHtml`) correctly contained "Message BBBB" — the flow received the right data.
5. **Result on the actual OneNote page**: the "Automated update" block was appended (confirming the append/match logic itself worked — same `MeetingId` was recognised as an existing page). But the block contains only the static notice text. "Message BBBB" does not appear anywhere on the page, before or after the update block.

## Root cause

`Compose_UpdateHtmlFragment` (in the existing-page/update branch of Flow B) is a **hardcoded static string**, not built from any trigger input:

```
<hr>
<h2>Automated update</h2>
<p><strong>Updated by:</strong> Meeting Capture Agent</p>
<p><strong>Update note:</strong> Meeting details were refreshed by the automation. Existing human-entered notes were preserved.</p>
```

It never references `triggerBody()?['text_3']` (the fresh `PageHtml` for this capture). `Update_page_content_Existing_Branch` then appends this fixed fragment (`action: append, position: after`) to the existing page — so the append mechanism itself is sound (proven by the earlier Bug 9 confirmation on 16 August, and re-confirmed here), but there was never any code path that carries the new meeting content into the appended block. This has presumably been true since `Compose_UpdateHtmlFragment` was first built — Bug 9's 16 August closure confirmed the *append plumbing* worked, but that test didn't check whether the new content was actually included (the test just verified original content survived and *a* block was appended).

## Impact

This affects **both** the recurring branch and the one-off branch equally, since `Compose_UpdateHtmlFragment` and `Update_page_content_Existing_Branch` are shared code used by both `Condition_Is_Genuine_Existing_Page`'s branches. So:
- One-off recapture (this test): confirmed broken.
- Recurring same-page update (e.g. today's "121 Simon / David" second capture, 18:06): almost certainly has the same gap — the Teams join-details block that appeared there was likely coincidentally similar-looking to real content, but is worth re-checking against this same root cause once the fix lands, since it may also have been the generic notice rather than genuine new content. *(Not yet independently confirmed — worth a quick check before or after the fix.)*

## Recommended fix

`Compose_UpdateHtmlFragment` needs to incorporate `triggerBody()?['text_3']` (or a suitably wrapped/sanitised version of it) into the appended block, rather than only the static notice. Draft shape:

```
<hr>
<h2>Automated update</h2>
<p><strong>Updated by:</strong> Meeting Capture Agent</p>
<p><strong>Update note:</strong> Meeting details were refreshed by the automation. Existing human-entered notes were preserved below.</p>
@{triggerBody()?['text_3']}
```

Open questions for build time:
- `text_3` currently arrives as a full HTML document (`<html><head><title>...</title></head><body>...</body></html>`), not a fragment — it likely needs the outer `<html>/<head>/<body>` tags stripped before insertion, similar to how the create-page path already just uses `<p>@{triggerBody()?['text_3']}</p>` directly (worth checking whether that path has the same double-wrapping issue, since `text_3` already contains its own nested `<html>` in the samples captured today — see the raw trigger bodies in this session).
- Confirm whether the timestamp/date of the update should be included in the notice line for clarity (e.g. "Meeting details were refreshed by the automation on 20 Aug 2026").

## Related items

- Confirms append/match plumbing (Bug 9, 16 Aug) is still sound — this is a distinct content-composition gap, not a regression of that fix.
- Relevant to the per-occurrence recurring-pages amendment (`design-amendment-2026-08-20-per-occurrence-recurring-pages.md`): once same-date recaptures append/protect on the recurring path too, they'll need this fix in place first, or they'll have the identical problem — organiser updates silently discarded there as well.

---
*Confirmed 20 August 2026 via live throwaway-meeting test (double capture, deliberate content edit between captures). Evidence: trigger raw bodies for both captures (identical `MeetingId`, differing `text_3`), and direct visual inspection of the resulting OneNote page content.*
