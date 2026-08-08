# INCIDENT — 8 August 2026 — PageWebUrl write causing repeated BadGateway + duplicate pages — RESOLVED (stability restored)

## Status: RESOLVED — flow stable and confirmed working. Underlying hyperlink fix remains outstanding (deliberately deferred, see below).

## Timeline

- Session resumed after mid-session power loss (see `handover-2026-08-08-URGENT-power-loss-midsession.md`).
- Restored to a clean pre-corruption base (Flow Checker 0 errors), then re-applied all four hyperlink-fix changes **one at a time with Flow Checker verification after each** — all four confirmed clean individually (0 errors, 1 pre-existing unrelated "Get items" OData filter warning throughout).
- **Published** the flow with all four changes live.
- First live capture attempt (~11:20) **failed**: `HTTP_Update_SP_PageSelfUrl` → `BadGateway`, after 6 automatic retries (per Power Automate's built-in retry policy), all failing.
- User resubmitted/recaptured multiple times (11:23, 11:26, 11:28, 11:30) — **every single attempt failed identically**, always the same action, always `BadGateway`.
- **28-day run history confirmed 5 consecutive failures in a row**, all on this same action, starting right after the hyperlink-fix publish.
- **Side effect discovered**: because `Create_OneNote_Page` runs and succeeds *before* the failing SharePoint step in the sequence, every failed/retried run created a **new duplicate OneNote page** ("12 Aug 2026" appeared twice under "Mtg - SC Eng Leadership Weekly") even though the flow overall reported failure.

## Root cause — NOT CONFIRMED (deliberately not investigated further; see Next steps)

Not diagnosed in detail before deciding to revert (prioritised stopping active production impact over full root-cause analysis, given failures were repeating on every attempt and duplicate pages were actively accumulating). Circumstantial evidence strongly points to the newly-added `PageWebUrl` field in the write body:

- Failures began immediately after the hyperlink-fix publish, with no other change in between.
- The `PageWebUrl` value being written is `Create_OneNote_Page`'s `links.oneNoteWebUrl.href` — a long, URL-encoded string (contains `%20`, `%E2%80%93`-style encoded characters per examples seen earlier in this project) — a plausible candidate for breaking JSON body construction or exceeding some limit, though this is a hypothesis, not confirmed.
- Not ruled out: coincidental genuine SharePoint-side outage across all 5 attempts (less likely given the consistency and specificity of always failing at the same single action, but not impossible).

**Still needs proper investigation in a calmer follow-up session** — inspecting the actual HTTP response body from a failed retry attempt (not just the `BadGateway` summary label) would be the right next step.

## Action taken — reverted write-side changes only

- **Recurring write point** (`HTTP Update SP PageSelfUrl`): body reverted from including `PageWebUrl` back to `PageSelfUrl`-only. Confirmed via Peek Code.
- **One-off write point** (`OF09b`): same revert. Note: first revert attempt did not save correctly (Peek Code still showed `PageWebUrl` present) — caught and corrected on second attempt, confirmed via Peek Code.
- Flow Checker after both reverts: **0 errors, 0 warnings** (even the pre-existing "Get items" warning cleared, which is fine/cosmetic).
- **Published.**

**Deliberately NOT reverted** (safe, left in place):
- The two **read-point** changes (`Compose_ExistingPageSelfUrl`, `OF02` reading `PageWebUrl` instead of `PageSelfUrl`) — harmless; with the write-side reverted, `PageWebUrl` is simply empty going forward, same as it was for all pre-existing rows.
- The `PageWebUrl` SharePoint column itself.

## Verification — CONFIRMED RESOLVED

- **Flow Checker**: 0 errors, 0 warnings, post-revert, post-publish.
- **Live test**: captured a recurring meeting (12 Aug occurrence) end-to-end — **succeeded**, no `BadGateway`, no failure. SharePoint mapping row correctly shows `PageSelfUrl` populated, `PageWebUrl` blank (expected, correct current-state behaviour).

**The flow is confirmed stable and safe for normal use as of this session.**

## What remains open (deliberately deferred, not urgent)

1. **The original hyperlink-truncation problem is NOT fixed** — links returned to users are still the long `PageSelfUrl` API reference, not a clickable web link. This reopens the issue from `handover-2026-08-06-oneNoteWebUrl-link-truncation-future-build.md` — treat that as still outstanding. Any future attempt at this fix should start by properly diagnosing today's `BadGateway` root cause first (see below) before re-attempting the write-side change.
2. **Root cause of the `BadGateway` still unconfirmed** — needs a calm, dedicated session: inspect actual SharePoint response body from a failed attempt (not just the summary label), and consider isolating whether it's genuinely the `oneNoteWebUrl` value's length/encoding, or something else, before redesigning the fix.
3. **Duplicate OneNote pages from this incident still need manual cleanup** — at minimum the duplicate "12 Aug 2026" page under "Mtg - SC Eng Leadership Weekly"; there may be others depending on which meetings were resubmitted during the 11:20–11:30 failure window. Recommend a manual check across recently-touched sections before assuming this is the only one.
4. **Bug 5** (one-off recapture, empty `sectionId`) — untouched today, still open.
5. **Microsoft support ticket** for the earlier corruption pattern — still not drafted; today's mid-session corruption recurrence (documented in the power-loss handover) is a further data point worth citing when it is.

## Status

**Flow stable, published, confirmed working via live test and clean Flow Checker. Bug 7 fix intact and unaffected. Hyperlink truncation issue reopened/still outstanding — do not re-attempt without first diagnosing today's BadGateway root cause.**
