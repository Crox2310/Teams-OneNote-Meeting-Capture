# Handover — 15 August 2026 (session 2, part 3) — Bug 9 deep trace, session closeout

## ⏭ START HERE

This is the final handover for an extremely long session on 15 August 2026. It covers the continuation from `handover-2026-08-15-session2-part2-recovery-complete-published.md` through to the end of the Bug 9 investigation. **Bug 9 is NOT fixed. The full expression chain has been verified correct, but the bug still reproduces — this is now suspected to be a new manifestation of the platform corruption pattern, affecting data values mid-flight rather than expressions or Flow Checker state.** Read this fully before resuming work on Bug 9.

---

## Session summary (this part)

Starting point: flow published, clean (0 errors), Bug 8 confirmed fixed, Bug 5's original symptom confirmed fixed, Bug 9 identified with an initial (later superseded) root-cause theory.

### Bug 9 investigation — full arc

**Attempt 1 (superseded)**: theorized the SharePoint `SeriesMasterId` "Enforce unique values" constraint was blocking one-off inserts, based on a "Required info" flag seen in the SharePoint UI. Changed `Enforce unique values` to `No` on `SeriesMasterId`. **Retested — Bug 9 still reproduced identically.** This fix did not address the actual cause, though it may still be a reasonable schema correction to keep (one-off rows have no `SeriesMasterId` by design, so uniqueness enforcement across blank values was never going to behave correctly).

**Attempt 2 (real, partial fix)**: traced `Update page content Existing Branch`'s raw inputs and found `pageId` contained a URL-encoded OneNote client deep-link (from the `oneNoteWebUrl` links object) instead of a proper API page reference (the `self` field). Found that `Compose_ExistingPageSelfUrl` (recurring branch) and `OF02 — Compose_ExistingPageSelfUrl_OneOff` (one-off branch) were both reading SharePoint's `PageWebUrl` column instead of `PageSelfUrl`. **Fixed both expressions** (`PageWebUrl` → `PageSelfUrl`). Flow Checker confirmed clean after each isolated save.

**Attempt 3 (real, partial fix, with a notable anomaly)**: retested — Bug 9 still reproduced identically despite the fix above. Traced downstream to `OF05a — Set varFinalExistingPageSelfUrl (OneOff)`, which on first inspection had **no `value` field at all** (only `name`) — meaning it never actually propagated `OF02`'s corrected output. **Before any edit was made to fix this, a second check of the same action showed the value field present and correct** (`@outputs('OF02_—_Compose_ExistingPageSelfUrl_OneOff')`), with no user edit made in between. This is a new corruption variant — not a value going blank, but a value appearing absent on one read and correct on the next, unprompted. Logged as a new data point for the Microsoft ticket.

**Attempt 4 (dead end — full chain verified correct, bug still reproduces)**: retested once more with a completely fresh MeetingId (`bug9-finalconfirm-15aug-1`). **Bug 9 still reproduced identically.** Checked every single action in the read/write chain, one at a time, in real time, immediately before/after checking:

1. `Create_OneNote_Page`'s raw output — `self` field confirmed correct (proper API page reference).
2. `Compose_PageSelfUrl_Created` — confirmed correct (`@body('Create_OneNote_Page')?['self']`).
3. `HTTP_Update_SP_PageSelfUrl` (non-OneOff, since this page went through the standard create path) — confirmed correctly wired to write `PageSelfUrl` from `Compose_PageSelfUrl_Created`'s output.
4. `OF02 — Compose_ExistingPageSelfUrl_OneOff` — confirmed still correctly reading `PageSelfUrl` (fix from Attempt 2 held).
5. `OF05a — Set varFinalExistingPageSelfUrl (OneOff)` — confirmed correct.
6. `Set_varOutputPageSelfUrl_Existing` — confirmed correct.
7. `Compose_ExistingPageId` — confirmed correct (`@last(split(variables('varOutputPageSelfUrl'), '/'))`).

**Every single action in this chain, verified in real time, is correct.** Yet:
- The `RecurringMeetingSectionMap` SharePoint list, checked directly, shows `PageSelfUrl` for the `bug9-finalconfirm-15aug-1` row **still contains the wrong (deep-link-style) value**, not a proper API reference.
- The corresponding flow run's `Update page content Existing Branch` action still received the wrong `pageId`, and still failed with `NotFound`.

**This is no longer explainable as a logic/expression bug.** Every expression inspected is correct. The only remaining explanation is that something is corrupting the *value* as it moves through the chain — between a correct action executing and its output being durably written or subsequently read — separate from and in addition to the previously-documented corruption patterns (blank SetVariable/InitializeVariable fields, publish-only validation gaps, self-resolving missing-value fields).

## New finding for the Microsoft ticket

**Corrupted data can propagate through a verified-correct expression chain.** This session confirmed, action by action, in real time, that every expression from OneNote page creation through to the final `pageId` composition is logically correct — yet the actual stored SharePoint value and the actual runtime `pageId` were both wrong, consistently, across three separate test cycles. This is a materially different and more concerning class of symptom than anything logged previously: it suggests the corruption isn't only a Designer-UI/save-time phenomenon affecting field definitions, but can also affect **data as it flows through a live, correctly-defined run** — which the existing "blank value + structural edit" mechanism hypothesis does not explain.

This should be added to the Microsoft ticket as a fifth distinct symptom category, alongside: (1) blank SetVariable/InitializeVariable values, (2) the "Invalid parameters" error variant, (3) the publish-only validation gap, (4) the self-resolving missing-value-field anomaly on `OF05a`, and now (5) correct-expression-chain-but-wrong-runtime-value.

## Status of all bugs at end of session

- **Bug 7** (recurring second-capture): fixed, not re-tested this session part.
- **Hyperlink truncation fix**: fixed, not re-tested this session part.
- **Bug 8** (`varOutStatus` empty): confirmed fixed via successful functional test earlier this session.
- **Bug 5** (one-off recapture, empty `sectionId`): original symptom confirmed fixed.
- **Bug 9** (`NotFound` on `Update page content Existing Branch`): **UNRESOLVED.** Two genuine partial fixes applied (`PageWebUrl`→`PageSelfUrl` on both branches' Compose actions; `SeriesMasterId` uniqueness constraint relaxed). Full expression chain verified correct end-to-end. Bug still reproduces. Root cause now suspected to be a new, data-level manifestation of the platform corruption pattern rather than a logic error. **No further fix attempted or recommended without first involving Microsoft, given the extensive verification already performed.**
- **Microsoft support ticket**: drafted and updated earlier this session with incidents 1–6; **needs one more update** to add the Bug 9 saga and the new "corrupted data through a verified-correct chain" finding before submission. Not yet submitted — David plans to submit later in the week.

## Immediate next steps for next session

1. **Do not attempt further live fixes to Bug 9 without first re-verifying current state from scratch** — given how many times values were confirmed-then-found-wrong-then-confirmed-again this session, treat nothing as stable until freshly checked.
2. **Update and submit the Microsoft ticket**, incorporating this session's full Bug 9 trace as supporting evidence — it's some of the most rigorous, systematic evidence gathered yet (a fully-verified-correct expression chain producing a wrong runtime result is unusual and worth Microsoft's direct attention).
3. Consider whether a full solution export/import (editing the flow definition outside the live Designer surface, as raised earlier in the ticket draft) might be worth trying specifically for Bug 9, since it may bypass whatever save-time or run-time process is corrupting this particular data path.
4. If Bug 9 persists after Microsoft's response, consider a structurally different mitigation: derive `pageId` directly from a fresh `Get Sections`/page-listing call at update time (matching by title, similar to how the section itself is resolved) rather than relying on any SharePoint-stored `PageSelfUrl` value at all — this would sidestep the entire fragile propagation chain, at the cost of an extra API call per update.

---

**Status: flow published and stable for its core functions (Bug 7, hyperlink fix, Bug 8, Bug 5's original symptom all working). Bug 9 remains open, extensively diagnosed, not resolved. Session closed here after exhausting straightforward debugging avenues.**
