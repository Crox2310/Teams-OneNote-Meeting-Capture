# URGENT handover — 15 August 2026 — mid-session, running out of image capacity, corruption pattern now confirmed reproducible

## ⏭ START HERE — READ BEFORE TOUCHING THE FLOW

David is about to run out of image-upload capacity in this chat. Session is paused mid-task. **Flow is currently in a RESTORED CLEAN STATE** (restored from Version History to the 8 August 12:05 PM version — confirmed via Flow Checker: 0 errors, 1 pre-existing harmless "Get items" OData warning). **This clean version does NOT yet have the Bug 8 fix (`varOutStatus = "OK"`) applied** — it still reads `""`, exactly as it did before this session started. Confirm current Flow Checker state before doing anything else, since this flow has been extremely volatile today.

---

## Critical new finding this session: corruption is now CONFIRMED REPRODUCIBLE, not random

**This is the most important thing to carry forward.** Editing `Set_varOutStatus`'s value field (changing `""` to `OK`) and saving has now triggered the platform-level corruption pattern (`SetVariable`/`InitializeVariable` value fields going blank, showing as 21–26 "Value is required" Flow Checker errors across unrelated actions) **three separate times**, across two different days (8 August and 15 August), with the same result every time. This is no longer "corruption happens sometimes" — it's a specific, repeatable trigger tied to editing this one action.

Additionally, this session found something more serious than previous occurrences: **a published flow can itself be silently corrupted.** Publishing does NOT run Flow Checker validation — they are completely independent. We published a version, got a green "success" banner, then ran Flow Checker separately and found 25 errors underneath. This means the corruption can reach production, not just the editing draft — and this may retroactively explain **Bug 5** (one-off recapture, empty `sectionId`): it's plausible Bug 5 isn't a separate logic bug at all, but this exact corruption pattern silently wiping `varTargetSectionPagesUrl` in production between sessions. Worth investigating together next time rather than treating them as unrelated.

## What was attempted this session (Bug 8 fix)

Goal: fix `Set_varOutStatus`, which is hardcoded to `""` instead of the historically-correct `"OK"` (see `handover-2026-08-08-bug8-outstatus-empty-new-section.md` for original diagnosis — confirmed via past chat history that this field's correct value was `"OK"` as of 27 July, and somewhere since then got silently wiped, exactly matching the corruption signature).

**Every attempt to make this one-line edit today triggered corruption:**
1. First attempt (early in session) — edited, corruption appeared, restored via Version History, redid the edit successfully that time, but then corruption struck again on a *later, untouched* screen navigation (zero-edit corruption).
2. Restored again to a clean version, re-attempted the edit, corrupted again.
3. Restored again to 8 August 12:05 PM (confirmed clean), attempted the edit a third time — corrupted again, identically.

**Decision made: stop attempting this fix via normal Designer editing for now.** Restored to the clean, pre-fix version and left `Set_varOutStatus` untouched. Bug 8 remains unfixed. This is the right call — better to have a stable, working flow (with Bug 7 and the hyperlink fix intact) than to keep risking corruption chasing one more fix.

## What was in progress when the session paused

David and Claude were in the process of **capturing a full backup of all ~26 known-good values** from the confirmed-clean 8 August 12:05 PM restore, specifically so that if corruption strikes again, the values can be quickly re-entered from a saved reference rather than needing another Version History hunt. **This was NOT completed** — only the list of which 26 actions to capture was established; the actual Peek Code values were not yet gathered before running out of image capacity.

### The 26 actions to capture (in flow order), for next session to pick up:

1. `varTargetSectionPagesUrl` (InitializeVariable, first/recurring instance)
2. `varOneNoteResolverResult` (InitializeVariable, first instance)
3. `varTargetSectionPagesUrl` (second/one-off instance)
4. `varOneNoteResolverResult` (second instance)
5. `Set varPageAction Created`
6. `Set varOutputPageSelfUrl Created`
7. `Set varOutputPageLink Created`
8. `Set varPageAction Created OneOff`
9. `Set varOutputPageSelfUrl Created OneOff`
10. `Set varOutputPageLink Created OneOff Gate`
11. `Set varPageAction ExistsNoCreate`
12. `Set varOutputPageSelfUrl Existing`
13. `Set varPageAction UpdatedAppend`
14. `Set varOutputPageLink Existing`
15. `Set varOutputPageLink Created OneOff`
16. `varFinalExistingPageSelfUrl` (first instance)
17. `varFinalPageDecision` (first instance)
18. `varFinalMatchCount` (first instance)
19. `Set varTargetSectionPagesUrl OneOff Exists`
20. `Set varOneNoteResolverResult Exists OneOff`
21. `Set varTargetSectionPagesUrl OneOff Created`
22. `Set varOneNoteResolverResult Created OneOff`
23. `OF05a — Set varFinalExistingPageSelfUrl (OneOff)`
24. `OF05b — Set varFinalPageDecision (OneOff)`
25. `OF05c — Set varFinalMatchCount (OneOff)`
26. `Set varOutStatus` — **NOTE: on the current clean restore point this correctly/expectedly reads `""`, not `"OK"` — this is the one value that needs to come from a DIFFERENT source (historical record) since the fix hasn't been safely applied yet.**

**Recommended for next session:** consider doing this capture via `Code view` full export rather than Peek Coding each action individually via screenshot — likely far fewer round trips and less risk of missing one. Worth checking if this flow's Designer offers a full-flow JSON export/download option.

## Recommended approach for next session on Bug 8

Given the edit-via-Designer-field approach has failed identically three times, **don't repeat it a fourth time without a different strategy.** Options worth considering, in rough order of promise:
1. **Full flow code export/import** (if available) — edit the JSON directly outside the Designer's live-editing surface, then re-import, avoiding whatever specifically triggers corruption during in-Designer field saves.
2. **Contact Microsoft support first**, now that we have a precise, three-times-reproduced repro case, and ask specifically about this pattern before attempting a fourth manual fix — this is now the strongest argument yet for finally prioritising that ticket (still not drafted, now overdue across multiple sessions: 1 Aug, 8 Aug, 15 Aug).
3. If attempting again manually: try editing in a completely fresh browser session/tab, freshly authenticated, in case cached state or a stale connection is contributing.

## Status of core fixes (unaffected by this session's problems)

- **Bug 7** (recurring second-capture) — still fixed, was live before this session, unaffected.
- **Hyperlink truncation fix** — still fixed, was live before this session, unaffected.
- **Bug 8** (`varOutStatus` empty, false failure message on new-section captures) — still unfixed. Root cause understood, fix is a single trivial value change, but blocked by corruption on every attempt.
- **Bug 5** (one-off recapture, empty `sectionId`) — still unfixed, and now suspected to possibly be the SAME corruption pattern rather than a separate bug — worth investigating together next session.
- **Microsoft support ticket** — still not drafted, now with three dated, detailed corruption incidents to cite, including one precise reproducible trigger case. Increasingly the top priority.

## Status

**Flow currently restored to a clean, stable, published-safe state matching 8 August 12:05 PM (Bug 7 + hyperlink fix intact, Bug 8 NOT applied). Confirm this is still the current state at the start of next session before doing anything else. Value-capture backup task incomplete — resume from the list above if attempting Bug 8 again, but consider the alternative strategies above first given three consecutive failures via the same method.**
