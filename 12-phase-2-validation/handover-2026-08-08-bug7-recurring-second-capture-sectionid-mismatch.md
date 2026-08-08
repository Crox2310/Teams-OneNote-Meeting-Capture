# Bug 7 — Recurring meeting second-capture fails with BadRequest (sectionId format) — FIXED AND LIVE-CONFIRMED

## Status: FIXED, published, live-tested successfully

## Reported by David, 8 August 2026, live in production use (not a test session)

**Symptom:** Capturing a recurring meeting for the first time works correctly. Capturing the *same* recurring meeting series again the following week — i.e. the second, third, etc. occurrence — fails. Teams shows a generic error to the user (`FlowActionBadGateway`); the underlying flow run shows `Flow run failed`.

**Confirmed across two independent recurring meetings** on 8 August (different meetings, same failure point both times) before the fix — this was a systemic bug on this path, not a one-off fluke.

## Root cause investigation (full trail, including a dead end — kept for the record)

Traced live via Activity → Run results → Code view on the failing action, `Update_page_content_Existing_Branch` inside `Apply_to_each_Existing_Section`.

**First hypothesis (SF-B7-01, tried and disproven):** the failing call sent `sectionId: 1-2175c4ee-...` (from `items('Apply_to_each_Existing_Section')?['id']`) alongside `pageId: 1-...!20-2175c4ee-...` (from stored `PageSelfUrl`) — same GUID, different numeric prefix (`1-` vs `20-`). Hypothesised the prefix mismatch was the cause; changed `sectionId` to parse the `20-...` reference out of `pageId` instead. **This did not fix it** — identical error persisted. Confirmed via a fresh `GetSectionsInNotebook` call that the section's real `id` genuinely is `1-2175c4ee-...` — so the original `1-` value was already correct as an ID; the prefix theory was wrong. Neither prefix variant was ever going to work, because the parameter wasn't expecting a bare ID in either format.

**Actual root cause (SF-B7-02, confirmed working):** the OneNote connector's `sectionId` parameter — despite its name — expects a **`pagesUrl`-style URL**, not a bare section ID string. Confirmed by cross-referencing the flow's own proven-working precedent: `Create_Page_OneOff` (live since 2 August) passes `variables('varTargetSectionPagesUrl')` — itself always populated from genuine `pagesUrl` fields elsewhere in the flow — into its own `sectionId` parameter, and that call succeeds. The section object returned by `GetSectionsInNotebook` includes a `pagesUrl` field alongside `id`/`self` (e.g. `https://www.onenote.com/api/v1.0/.../notes/sections/{id}/pages`); this is what the parameter actually wants.

## Fix applied — SF-B7-02

**`Update_page_content_Existing_Branch`**, `sectionId` parameter, changed to:
```
@items('Apply_to_each_Existing_Section')?['pagesUrl']
```
(previously `@items('Apply_to_each_Existing_Section')?['id']`, then briefly the SF-B7-01 parse-from-pageId expression — both superseded by this fix)

Verified via Peek Code before publishing, matched intended expression exactly. Flow Checker clean. Published.

## Live test result — SUCCESS

Captured a second occurrence of "SC Eng Leadership Weekly" (previously-failing meeting). Teams returned: *"Great news! Your meeting notes for SC Eng Leadership Weekly have been saved to OneNote."* No error, no BadRequest. **First confirmed live success on the recurring second-capture / update-existing-page path.**

## Separately noted during live test — NOT a regression, already logged elsewhere

Clicking the returned link produced a `C40001` "does not contain a valid authentication token" error in the browser. This is the **pre-existing, already-documented link-truncation issue** from `handover-2026-08-06-oneNoteWebUrl-link-truncation-future-build.md` — `varOutputPageLink` on this branch is still sourced from the raw OneNote API self-reference URL (`PageSelfUrl`) rather than the proper `oneNoteWebUrl` web link, so it isn't browser-openable without an auth token. This is unrelated to and unaffected by the Bug 7 fix — the page itself was genuinely created/updated correctly; only the returned *link* is the wrong kind. No new work needed here beyond the fix already scoped in the 6 August doc.

## Relationship to other known issues

- **Distinct from Bug 5** (one-off recapture, empty `sectionId` string) — different symptom, different branch, different root cause. Bug 5 remains open.
- **Confirms and reinforces** the 6 August link-truncation finding — this session's live test is now direct evidence (not just theoretical) that the same page reliably produces a working *content update* but a broken *returned link*, strengthening the case for that fix.

## Priority note for remaining open items

With Bug 7 now fixed, the highest-impact remaining item is arguably the **hyperlink fix**, since it's now confirmed to reproduce on every successful capture (not hypothetical) — every user who successfully captures a meeting currently gets a broken link. Bug 5 (one-off recapture) remains open and lower-frequency by comparison.

## Status

**Fixed, published, live-tested successfully against a real previously-failing recurring meeting.** Open items remaining: Bug 5 (one-off recapture), hyperlink fix (now confirmed reproducing live), Microsoft support ticket for the earlier corruption pattern (still outstanding, unrelated to this bug).
