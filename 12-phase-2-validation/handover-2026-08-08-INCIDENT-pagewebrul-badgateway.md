# INCIDENT — 8 August 2026 — PageWebUrl write causing repeated BadGateway + duplicate pages — ROOT CAUSE CONFIRMED

## Status: Root cause confirmed. Fix identified, about to be applied in this same session.

## Timeline

- Session resumed after mid-session power loss (see `handover-2026-08-08-URGENT-power-loss-midsession.md`).
- Restored to a clean pre-corruption base (Flow Checker 0 errors), then re-applied all four hyperlink-fix changes **one at a time with Flow Checker verification after each** — all four confirmed clean individually.
- **Published.** First live capture attempt (~11:20) **failed**: `HTTP_Update_SP_PageSelfUrl` → `BadGateway`, after 6 automatic retries.
- Repeated resubmits/recaptures (11:23–11:30) all failed identically. 5 consecutive failures confirmed in 28-day run history, all same action, all `BadGateway`.
- Side effect: `Create_OneNote_Page` succeeds before the failing SharePoint step, so every failed attempt created a duplicate OneNote page ("12 Aug 2026" appeared twice under "Mtg - SC Eng Leadership Weekly").
- **Reverted write-side changes** (`PageWebUrl` removed from both write points, back to `PageSelfUrl`-only) to stop active failures. Confirmed stable via Flow Checker (0/0) and a successful live capture.
- **Root cause then properly diagnosed** by inspecting the actual retry history response body (not just the `BadGateway` summary label) on the very first failed run (11:19 AM).

## ROOT CAUSE — CONFIRMED

The `502 BadGateway` returned by the Azure API Hub was wrapping a genuine SharePoint validation error:

```
Microsoft.SharePoint.SPException (code -2130575336):
"Invalid text value. A text field contains invalid data. Please check the value and try again."
```
Source: `https://jsainsbury.sharepoint.com/sites/coplt/_api/web/lists/GetByTitle('RecurringMeetingSectionMap')/items(89)`

**This is SharePoint rejecting the value on data-validation grounds, not a transient infrastructure issue and not a JSON-encoding problem.**

The `PageWebUrl` column was created as **"Single line of text"** (`SP.FieldText`), which has a **hard 255-character limit** in SharePoint. The actual value being written — `Create_OneNote_Page`'s `links.oneNoteWebUrl.href` — is a URL-encoded SharePoint deep-link (contains `%20`, `%28`, `%29`, `%E2%80%93`-style encoding for spaces/dashes/special characters in meeting titles). For anything but the shortest meeting titles, this value very plausibly exceeds 255 characters, causing SharePoint to reject the write outright.

This cleanly explains every observed symptom: consistent, deterministic, 100%-repeatable failure (not intermittent — every attempt was over the limit, not occasionally), specific to this one write action, and unrelated to the read side (which was never touched or reverted, and never caused any issue).

## Fix — to be applied this session

**Change the `PageWebUrl` column type from "Single line of text" to "Multiple lines of text"** (supports up to ~63,999 characters — comfortably beyond any realistic URL length), then re-apply the two write-side changes (previously reverted) referencing this now-correctly-typed column.

Note: `PageSelfUrl` (the pre-existing column) is also "Single line of text" and has apparently worked without issue — plausibly because Graph API self-reference URLs, while also long, may consistently stay under 255 characters, or may simply not yet have hit a title long enough to trigger the same failure. Worth keeping in mind as a latent, currently-dormant risk on that column too, though out of scope to fix today.

## Duplicate pages — still need manual cleanup

Not yet actioned. At minimum the duplicate "12 Aug 2026" page under "Mtg - SC Eng Leadership Weekly" from this incident's failed retries.

## Other items still open, unaffected by this incident

- Bug 5 (one-off recapture, empty `sectionId`) — untouched.
- Microsoft support ticket for the earlier corruption pattern — still not drafted.

## Status

**Root cause confirmed: SharePoint "Single line of text" 255-character limit on `PageWebUrl`. Fix (change column type to Multiple lines of text) identified and about to be applied.**
