# Handover — 16 August 2026 (session close) — Express mode setting will not stay off; second corruption event confirmed; Delay-based fix abandoned for now

## START HERE

This is the closing handover for today's session, continuing directly from `handover-2026-08-16-express-mode-corruption-trigger-recovered.md`. That handover documented one corruption event triggered by toggling Express mode off. **This session confirmed the pattern is worse than a one-off: Express mode will not stay off even after an explicit toggle-and-save, and re-triggered the same 26-action corruption a second time.** The Delay-based reliability fix for `Set_PageTitle_Recurring` has been abandoned for today as a result, and the flow has been restored to a clean, stable state without it.

**This is now a confirmed, repeatable finding, not a single anomalous incident — treat it as such in the Microsoft ticket.**

---

## What happened, in sequence

1. Following the first Express-mode corruption event and full 26-action manual recovery (see previous handover), David attempted to toggle Express mode off again via the Overview/Details settings panel, to unblock the `Delay_Before_TitleSet_Recurring` action.
2. **Express mode reverted itself back to Enabled after being explicitly toggled off and saved** — observed directly, not inferred. No further Designer edits were made in between the save and the setting reverting.
3. This produced a repeat of the exact same conflict as before: a red banner reporting a conflict between the (still-present) Delay action and Express mode being back on.
4. **Flow Checker jumped to 26 errors again** — the identical signature (blanked `value` fields across the same page-creation, mapping-write, and section-resolution `SetVariable` actions) as every prior incident this week, now confirmed to occur on this specific trigger **twice in immediate succession**.
5. David manually restored all 26 values a second time, following the same one-at-a-time, save-and-verify-each discipline as the first recovery.
6. **This time, `Delay_Before_TitleSet_Recurring` was deliberately not recreated** during the restore — since Express mode would not reliably stay off, there was no point re-adding an action that requires it to be off to function. This incidentally also removes the direct trigger for the Express-mode/Delay conflict specifically, even though the deeper mystery (why the setting won't hold) remains unexplained.
7. Flow Checker confirmed clean (0 errors) after this second recovery.

## Why this matters more than the first incident

A setting reverting itself after an explicit save, independent of any Designer canvas edit, is a materially different and more concerning class of problem than the earlier structural-corruption incidents:

- Every previous corruption event (Designer edits, publish events, and now the first Express-mode toggle) involved a **user-initiated action** that could at least in principle be avoided or sequenced around.
- **A setting that reverts itself after being saved, with no further action taken, is not something the user can work around through discipline or careful sequencing.** This is the first finding this week that isn't mitigated by "make isolated changes, verify after each, don't leave fields blank."
- This should be flagged to Microsoft as a distinct, high-priority symptom, separate from (though possibly related to) the value-blanking corruption pattern already documented.

## Current state

- Flow Checker: 0 errors, confirmed clean after second full manual recovery.
- **`Delay_Before_TitleSet_Recurring` no longer exists in the flow** — deliberately not recreated, given Express mode's instability.
- **The recurring-branch title-set reliability issue (the `404`/OneNote-error-20102 race condition on `Set_PageTitle_Recurring`) remains unresolved.** The Delay-based fix approach has been abandoned for now, not because it was proven wrong, but because the platform's Express-mode behaviour made it impossible to test or deploy safely today.
- Express mode's actual current on/off state should be re-verified at the start of next session — do not assume it has held in either direction.

## Recommended next steps

1. **Add both Express-mode findings to the Microsoft support ticket as high-priority items**: (a) toggling Express mode off can trigger the same mass value-blanking corruption as other structural changes, and (b) the setting has been observed reverting itself to Enabled after an explicit save, independent of further user action — confirmed twice.
2. **Do not attempt the Delay-based fix again until Express mode's behaviour is understood or Microsoft has responded.** If the reliability fix is still wanted, consider an alternative approach that doesn't require Express mode to be off at all — e.g. the "Do until" retry/poll pattern discussed earlier today as Option B's alternative, which uses ordinary HTTP-call latency rather than an explicit Delay/Wait action, and may not trigger this specific incompatibility.
3. Re-verify Flow Checker and Express mode state at the very start of next session before any other work, given this setting's demonstrated instability.
4. All other items from earlier today remain open and unaffected: the one-off branch's stale-mapping edge-case test, the unrelated tail-section anomaly, and eventually reverting the Bug 9 workaround to genuine title-matching.

---

**Status: flow stable, Flow Checker 0 errors, Delay-based fix abandoned and removed from the flow. Two consecutive Express-mode-triggered corruption events fully recovered from via manual re-entry, no data or functionality lost. This is a good, deliberate stopping point for today — the platform's current behaviour around this specific setting is not something to keep pushing against in the same session.**
