# FIX LOG — #2 append-content-loss resolved (21 August 2026)

## Status
**Fixed, deployed, and confirmed working via live recapture test. No further action needed.**

## What was fixed
`Compose_UpdateHtmlFragment` (Flow B, existing-page/update branch) was a hardcoded static
string that never referenced `triggerBody()?['text_3']` — so a recapture correctly preserved
original page content but silently discarded the organiser's actual updated content, appending
only a generic "refreshed" notice. See `bug-2026-08-20-update-fragment-discards-new-content.md`
for the original diagnosis.

## The fix applied
`Compose_UpdateHtmlFragment`'s expression changed from the static string to:
```
@concat('<hr><h2>Automated update</h2><p><strong>Updated by:</strong> Meeting Capture Agent</p><p><strong>Update note:</strong> Meeting details were refreshed by the automation. Existing human-entered notes were preserved below.</p>', triggerBody()?['text_3'])
```
This appends the notice plus the actual new capture content (`text_3`), reusing the same
payload the create path already sends to OneNote successfully — no `<body>` extraction or
substring surgery needed, per the dry-run reasoning in the original design session.

## Live confirmation (21 August 2026)
Test performed on the "Test Meeting 1" one-off used throughout this project's testing:
1. Baseline re-confirmed clean: one page, one SharePoint mapping row, content "Message AAAA".
2. Meeting body amended in Outlook: "Message AAAA" → "Message BBBB".
3. Recaptured.

**Result — all confirmed correct:**
- **Still exactly one page** in the section (no duplicate created) — confirms matching (`MeetingId`) continues to work correctly on recapture.
- **Still exactly one SharePoint mapping row**, same `MeetingId`, `Status: Active`.
- **Page content correctly shows, top to bottom:** original "Message AAAA" content (preserved) → "Automated update" notice → a second "Meeting Details" header → **"Message BBBB" (the actual new content, now genuinely appended)**.
- **No stray HTML/tag rendering junk** despite `text_3` being a full nested HTML document — the concern flagged in the original design session (that appending the raw `text_3` mid-page might render nested `<html>/<head>/<title>` tags as visible junk) did not materialise. OneNote handled it cleanly, same as it already did on the create path.

No fallback (the `<body>`-extraction substring approach, also designed and dry-run tested during the original session) was needed.

## Known separate issue re-confirmed present (not related to this fix)
The page-link click error (`C40001`, "does not contain a valid authentication token") occurred
again when testing — this is the pre-existing, already-logged link-format bug (flow returns the
raw OneNote API self-reference instead of `oneNoteWebUrl` in some outputs). Unrelated to #2;
noted here only because it appeared in the same test session's evidence.

## Repo/build status update
- **#2 (this fix): done.**
- **#3 (date handling): done** (see `fix-2026-08-20-3-datehandling-resolved.md`), including the
  C6D number-selection regression fix. FA16 Flow A defensive guard still pending (optional,
  defence-in-depth only).
- **#1 (per-occurrence recurring pages): design complete, not yet built.** Was explicitly
  sequenced after #2 in the original plan (per
  `design-amendment-2026-08-20-per-occurrence-recurring-pages.md`) because same-date recaptures
  on the recurring path would otherwise have inherited this exact bug. That dependency is now
  clear — #1 can proceed.

---
*Confirmed 21 August 2026 via live recapture test on "Test Meeting 1". Deployed by David
independently using the SF-03 expression provided in the prior session; verified against a
fresh evidence capture (raw trigger body confirming `text_3` content, OneNote page screenshot,
SharePoint mapping list) before being logged as closed.*
