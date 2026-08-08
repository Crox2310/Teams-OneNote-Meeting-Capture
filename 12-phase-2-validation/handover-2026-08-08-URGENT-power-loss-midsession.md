# URGENT handover — 8 August 2026 — mid-session power loss, DRAFT NOT PUBLISHED, corruption pattern reappeared

## ⏭ START HERE NEXT SESSION — READ BEFORE TOUCHING THE FLOW

**Status: David lost power mid-session. Flow is in a BROKEN DRAFT state (26 Flow Checker errors) which was NEVER PUBLISHED. The last PUBLISHED version is Bug 7's fix, which is safe, live, and confirmed working.** Do not assume today's later work (the hyperlink fix) is live — it is not. Check Version History first thing next session.

---

## What was achieved and IS safely published/live

**Bug 7 (recurring meeting second-capture BadRequest) — FIXED, PUBLISHED, LIVE-CONFIRMED.** Full detail in `12-phase-2-validation/handover-2026-08-08-bug7-recurring-second-capture-sectionid-mismatch.md`. Root cause: `Update_page_content_Existing_Branch`'s `sectionId` parameter needed a `pagesUrl`-style URL (`items('Apply_to_each_Existing_Section')?['pagesUrl']`), not a bare section ID. Fixed, published, and live-tested successfully against a real previously-failing recurring meeting ("SC Eng Leadership Weekly") this session. **This fix is safe regardless of what happens next session — it was published before the corruption appeared.**

---

## What was IN PROGRESS and is NOT published — the hyperlink truncation fix

Continuing from `12-phase-2-validation/handover-2026-08-06-oneNoteWebUrl-link-truncation-future-build.md`, this session began building the fix for the `PageSelfUrl`-vs-`oneNoteWebUrl` link truncation issue (confirmed live-reproducing during the Bug 7 test — every successful capture currently returns a broken, unclickable link).

### Completed this session (all in the current unpublished draft):

1. **SharePoint column added**: `PageWebUrl`, Single line of text, added to `RecurringMeetingSectionMap` list at `https://jsainsbury.sharepoint.com/sites/coplt`. **This is a genuine SharePoint list change and persists regardless of the flow's draft/publish state** — confirmed created correctly, no action needed here next session.

2. **Recurring write point** — `HTTP_Update_SP_PageSelfUrl` body changed from:
   ```json
   { "PageSelfUrl": "@{outputs('Compose_PageSelfUrl_Created')}" }
   ```
   to:
   ```json
   {
     "PageSelfUrl": "@{outputs('Compose_PageSelfUrl_Created')}",
     "PageWebUrl": "@{outputs('Create_OneNote_Page')?['body']?['links']?['oneNoteWebUrl']?['href']}"
   }
   ```
   Verified via Peek Code, correct.

3. **One-off write point** — `OF09b — HTTP Update SP PageSelfUrl (OneOff)` body. **This one had a false start, corrected within-session:**
   - First attempt wrongly referenced `Create_Page_OneOff` — this action is **not** in this branch's ancestry (`Create_Page_OneOff` only exists in the separate `Condition_Is_Genuine_Existing_Page` → False branch, Bug 5's territory) and caused a genuine `InvalidTemplate` / cross-branch-reference Flow Checker error on first Publish attempt.
   - **Root cause found**: `OF09b`'s branch shares the **same** `Create OneNote Page` action as the recurring path (both live under `Condition Should Create Page` → True, before `OF09-Gate` splits recurring vs one-off). Confirmed via Peek Code that `Compose_PageSelfUrl_Created` itself reads `@body('Create_OneNote_Page')?['self']` — proving this Compose action is shared, not recurring-specific.
   - **Corrected fix applied**: body changed to reference `Create_OneNote_Page` (not `Create_Page_OneOff`):
     ```json
     {
       "PageSelfUrl": "@{outputs('Compose_PageSelfUrl_Created')}",
       "PageWebUrl": "@{body('Create_OneNote_Page')?['links']?['oneNoteWebUrl']?['href']}"
     }
     ```
   - **A second mistake happened applying this**: editing the Body field via its inline fx/lightning-bolt icon replaced the *entire* body content rather than inserting into it, leaving invalid JSON (missing braces, missing `PageSelfUrl`, orphaned `PageWebUrl` fragment) for a period. **This was caught and corrected within-session** — final Peek Code confirmed correct, full valid JSON with both fields.

4. **Recurring read point** — `Compose_ExistingPageSelfUrl` changed field reference from `PageSelfUrl` to `PageWebUrl` only (structure otherwise untouched):
   ```
   @if(
     greater(length(body('Filter_Existing_Mapping')), 0),
     first(body('Filter_Existing_Mapping'))?['PageWebUrl'],
     ''
   )
   ```
   Verified via Peek Code, correct.

5. **One-off read point** — `OF02 — Compose ExistingPageSelfUrl OneOff` changed the same way:
   ```
   @if(greater(length(body('OF01_—_Filter_Existing_Mapping_OneOff')), 0), first(body('OF01_—_Filter_Existing_Mapping_OneOff'))?['PageWebUrl'], '')
   ```
   Verified via Peek Code, correct. Field reference inserted via the picker (not retyped) specifically to avoid the em-dash/hyphen corruption risk documented elsewhere in this flow's history.

**Deliberately NOT touched**: `varFinalExistingPageSelfUrl`, `varOutputPageSelfUrl`, `Compose_ExistingPageId` — these remain reading from the original SelfUrl chain, since they're used for genuine internal purposes (e.g. `Compose_ExistingPageId` derives the actual OneNote page ID, which Bug 7's fix depends on). Only the *user-facing link* output was re-pointed to `PageWebUrl`.

### What broke — the corruption pattern reappeared

After the above five changes were all individually verified correct via Peek Code, Flow Checker was run and returned **26 errors**, all pattern `'Value' is required` on `SetVariable`/`InitializeVariable` actions **we had not touched this session** — `varTargetSectionPagesUrl 1`, `varOneNoteResolverResult 1`, `varTargetSectionPagesUrl 2`, `varOneNoteResolverResult 2`, `Set varPageAction Created`, `Set varOutputPageSelfUrl Created`, and more (list continues, not fully captured before power loss).

This matches **exactly** the corruption signature from `handover-2026-08-01-corruption-incident-and-fix-list.md` and the session 4/5 handovers — value fields going blank simultaneously on unrelated actions, not a logical consequence of the actual edits made. **This is very likely the same unresolved platform-level corruption issue, not a mistake in the five changes listed above.**

**Correctly, the session stopped here — the draft was NOT published in this broken state.** David lost power shortly after this was discovered, before a decision was made on how to proceed.

---

## Recommended next session start (in order)

1. **Check Version History first, before touching anything.** Look for a restore point from earlier in this session — ideally right after the Bug 7 live-test success, or right after the `OF02` Peek Code confirmation (the last fully-clean checkpoint before the 26-error state appeared). Restoring is strongly preferred over manually re-patching 26 actions, per the hard lesson from the 1 August incident (manual re-patching led to fixes reverting and a wasted session).
2. **If a clean restore point exists**: restore to it, then re-apply the five hyperlink-fix changes above **one at a time, saving and re-verifying via Peek Code after each single change**, rather than making all five and checking once at the end — this matches the working discipline established after the 1 August incident.
3. **If no clean restore point exists** covering today's Bug 7 fix: the published (live) version already has Bug 7 safely fixed — worst case, restore to that published version and re-apply all of today's hyperlink work fresh, treating it as a new session.
4. **Once a clean base is confirmed**: re-apply changes 2–5 above (SharePoint column is already done and persists regardless), Flow Checker after each, then Publish, then live-test — capture a **new** meeting (not a pre-fix one, since old mapping rows won't have `PageWebUrl` populated) and confirm the returned link actually opens in browser without the `C40001` auth error.
5. Consider, once this is stable: whether today's corruption recurrence is further evidence to escalate to Microsoft support (per the still-outstanding, high-priority ticket noted in every session since 1 August) — this may be worth raising as a fresh, dated data point in that ticket once drafted.

## Other items still open, unaffected by today

- **Bug 5** (one-off recapture, empty `sectionId`) — still diagnosed, not fixed. Untouched today.
- **Microsoft support ticket** for the corruption pattern — still not drafted, now with a fresh (fourth+) occurrence to cite.

## Status

**Bug 7: fixed, published, live, safe.** **Hyperlink fix: fully designed and individually verified action-by-action, but blocked by a fresh corruption event before publish — currently sitting in a broken, unpublished draft state.** SharePoint `PageWebUrl` column exists and is safe. No data loss risk to production — the published flow is still Bug-7-fixed and functional; only the in-progress draft is currently broken.
