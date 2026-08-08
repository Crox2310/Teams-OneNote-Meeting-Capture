# INCIDENT — 8 August 2026 — PageWebUrl write causing repeated BadGateway + duplicate pages

## Status: ACTIVE INCIDENT, reverting write-side changes now

## Timeline

- Session resumed after mid-session power loss (see `handover-2026-08-08-URGENT-power-loss-midsession.md`).
- Restored to a clean pre-corruption base (Flow Checker 0 errors), then re-applied all four hyperlink-fix changes **one at a time with Flow Checker verification after each** — all four confirmed clean individually (0 errors, 1 pre-existing unrelated "Get items" OData filter warning throughout).
- **Published** the flow with all four changes live.
- First live capture attempt (~11:20) **failed**: `HTTP_Update_SP_PageSelfUrl` → `BadGateway`, after 6 automatic retries (per Power Automate's built-in retry policy), all failing.
- User resubmitted/recaptured multiple times (11:23, 11:26, 11:28, 11:30) — **every single attempt failed identically**, always the same action, always `BadGateway`.
- **28-day run history confirmed 5 consecutive failures in a row**, all on this same action, starting right after the hyperlink-fix publish.
- **Side effect discovered**: because `Create_OneNote_Page` runs and succeeds *before* the failing SharePoint step in the sequence, every failed/retried run created a **new duplicate OneNote page** ("12 Aug 2026" appeared twice under "Mtg - SC Eng Leadership Weekly") even though the flow overall reported failure. Duplicate pages will need manual cleanup once the flow is stable again.

## Root cause — NOT YET CONFIRMED

Not diagnosed in detail before deciding to revert (prioritised stopping active production impact over full root-cause analysis, given failures were repeating on every attempt and duplicate pages were actively accumulating). Circumstantial evidence strongly points to the newly-added `PageWebUrl` field in the write body:

- Failures began immediately after the hyperlink-fix publish, with no other change in between.
- The `PageWebUrl` value being written is `Create_OneNote_Page`'s `links.oneNoteWebUrl.href` — a long, URL-encoded string (contains `%20`, `%E2%80%93`-style encoded characters per examples seen earlier in this project) — a plausible candidate for breaking JSON body construction or exceeding some limit, though this is a hypothesis, not confirmed.
- Not ruled out: coincidental genuine SharePoint-side outage across all 5 attempts (less likely given the consistency and specificity of always failing at the same single action, but not impossible).

**This needs proper investigation in a calmer follow-up session** — inspecting the actual HTTP response body from a failed retry attempt (not just the `BadGateway` summary label) would be the right next step, ideally via Retry history or raw output inspection, once the flow is stable again and not under live-incident pressure.

## Immediate action taken — revert write-side changes only

To stop further failures and duplicate-page creation, reverting **only** the two write-point changes back to their pre-today state:

- **Recurring write point** (`HTTP Update SP PageSelfUrl`): body reverted from including `PageWebUrl` back to `PageSelfUrl`-only.
- **One-off write point** (`OF09b`): same revert.

**Deliberately NOT reverted** (safe to leave in place):
- The two **read-point** changes (`Compose_ExistingPageSelfUrl`, `OF02` reading `PageWebUrl` instead of `PageSelfUrl`) — harmless on their own; with the write-side reverted, `PageWebUrl` will simply be empty/never populated going forward, same as it already was for all pre-existing rows. No functional regression versus this morning's known state.
- The `PageWebUrl` SharePoint column itself — harmless to leave in the list unused.

This means: **the original hyperlink-truncation problem (links showing `PageSelfUrl` instead of a working web link) is NOT fixed by this revert** — it returns to the same state as this morning, before today's fix attempt. That's an acceptable trade-off given the alternative (repeated capture failures + duplicate pages) is actively worse.

## Next steps (separate, calmer session)

1. Confirm the revert actually stops failures — live-test one new capture end-to-end after publishing the reverted version.
2. Clean up duplicate "12 Aug 2026" pages created during the failed retries.
3. Properly diagnose why `PageWebUrl` in the write body caused `BadGateway` — inspect actual SharePoint response body from a failed attempt, not just the summary label. Consider testing with a shorter/simpler test value first to isolate whether it's genuinely the URL content/length, or something else entirely.
4. Once root cause is confirmed, redesign the write-side fix appropriately (e.g., URL-encoding/escaping considerations, checking SharePoint column data type limits, or testing whether the issue reproduces consistently in isolation) before re-attempting.
5. Re-apply the read-side is already fine; only the write-side needs a redesigned fix.

## Status

**Write-side reverting now. Hyperlink truncation issue reopened (was not successfully fixed today) — treat `handover-2026-08-06-oneNoteWebUrl-link-truncation-future-build.md`'s original issue as still outstanding.** Bug 7 (recurring second-capture) remains fixed and unaffected by this incident. Duplicate OneNote pages from this incident still need manual cleanup.
