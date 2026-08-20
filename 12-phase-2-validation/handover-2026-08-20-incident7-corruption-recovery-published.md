
# Handover — 20 August 2026 — Incident 7: corruption recovery, published clean

## Summary

While adding the OneOff-branch Delay action (mirroring the True-branch fix from 19 August, resolving the BadGateway/NotFound/FlowActionTimedOut race condition), Flow Checker suddenly showed **22 errors** across "PA - Resolve OneNote Meeting Section - v2 Clean Build" — the same recurring corruption pattern first documented on 15 August (Incidents 1–6). This is logged as **Incident 7**.

## Context leading up to it

This session had already:
1. Added and verified the second `Delay` action (5s) in the OneOff fallback branch, between `Create_Page_OneOff` and `Get_Pages_In_Section_OneOff_PostCreate` — confirmed correct via Code view (`type: Wait`, `count: 5`, `unit: Second`, correctly scoped `runAfter`).
2. Begun investigating the original open bug (Teams→OneNote link not displaying to the user) — traced to `outstatus` coming back blank on a run where `outonenoteresolverresult` was `CreatedSection` (new section had to be created before page creation). This matches the known **Bug 8** open item from the 15 August reference doc.
3. Corruption struck during this investigation, before the Bug 8 root cause could be confirmed.

## Recovery

- All 22 corrupted actions matched exactly against the existing `flow-reference-2026-08-15-full-peek-code-capture.md` reference table — same action names, same missing-`value` corruption signature as documented on 15 August.
- Values written back one at a time, using the reference doc as the source of truth (including the three INFERRED entries: `varFinalExistingPageSelfUrl_1`, `varFinalPageDecision_1`, `varFinalMatchCount_1`).
- Flow Checker reached **0 errors / 1 warning** (the standard "Get items" OData filter warning — expected/cosmetic, matches the known healthy baseline).
- Per the lesson learned on 15 August (Flow Checker clean ≠ Publish will succeed — see the publish-only validation gap finding in `handover-2026-08-15-session2-part2-recovery-complete-published.md`), the top-level `varTargetSectionPagesUrl` InitializeVariable action was checked directly via Code view before considering recovery complete. Confirmed clean this time — only `name` and `type` fields present, no stray out-of-scope `value` field (unlike the equivalent fault found on 15 August).
- **Publish attempted and succeeded** (green banner). This is the genuine confirmation point, not just a clean Flow Checker.

## Outcome

- Recovery time: notably faster than 15 August, entirely due to having the reference doc ready — no re-derivation of expressions needed.
- No new corruption sub-patterns observed this time (unlike Incidents 5/6, which introduced a new "Invalid parameters" error class). This looked like a straightforward repeat of the original missing-`value` signature.
- OneOff Delay action (this session's original goal) confirmed still present and correct after recovery — not affected by the corruption.

## Open items carried forward

1. **Bug 8 (`varOutStatus` blank on CreatedSection branch)** — investigation was interrupted by this incident, not resolved. This is the immediate next step post-recovery: confirm whether `Set_varOutStatus`'s `runAfter` (currently `Compose_SP_Item_Count: Succeeded`) is actually reached when the flow takes the CreatedSection path, or whether that branch dead-ends before reaching it.
2. **Microsoft support ticket** — still not submitted as of this incident. Now **seven** dated incidents. Should include this session's occurrence as further evidence of frequency/reproducibility.
3. Continue treating any edit as a potential corruption trigger — this incident occurred with a routine, correct single-action addition (the OneOff Delay), same as several of the 15 August incidents.

---

*Logged 20 August 2026. Cross-reference: `flow-reference-2026-08-15-full-peek-code-capture.md`, `handover-2026-08-15-session2-part2-recovery-complete-published.md`, `MICROSOFT-SUPPORT-TICKET-DRAFT-2026-08-15.md`.*
