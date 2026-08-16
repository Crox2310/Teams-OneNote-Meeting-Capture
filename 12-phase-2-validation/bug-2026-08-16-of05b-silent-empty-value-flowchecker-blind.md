# Bug/Corruption — `OF05b — Set varFinalPageDecision (OneOff)` value silently blanked — invisible to Flow Checker

**Found:** 16 August 2026, ~2:25 PM, live at work — second failed attempt to capture "NH Performance Feedback - David, Simon & Jin Connect" as a genuinely fresh one-off meeting (no SharePoint mapping row existed for it).
**Status:** root cause confirmed via live trace. Fix identified and applied by David. Supersedes the earlier same-day theory in `bug-2026-08-16-oneoff-stale-mapping-empty-sectionid.md` for *this specific run* — that bug is real and still valid for genuinely stale mappings, but was not the cause of this particular failure.

---

## What actually happened (corrected root cause)

Initial theory (logged earlier today) was that this meeting had a stale SharePoint mapping row pointing at a broken section. **That was ruled out** — SharePoint was confirmed empty for this meeting before this run.

Live trace instead showed:

- **`OF03 — Compose PageDecision OneOff`**: correctly computed and output `"PAGE_NOT_FOUND"` (correct — SharePoint has no mapping, so a new page should be created via the normal path).
- **`OF05b — Set varFinalPageDecision (OneOff)`**, the very next action, whose only job is `varFinalPageDecision = outputs('OF03_—_Compose_PageDecision_OneOff')`: its raw output showed `"value": ""` — **empty**, not `PAGE_NOT_FOUND`.

Because `varFinalPageDecision` ended up blank instead of `PAGE_NOT_FOUND`, `Condition_Should_Create_Page`'s check (`variables('varFinalPageDecision') == 'PAGE_NOT_FOUND'`) evaluated **False** — routing a genuinely brand-new one-off meeting into the wrong branch (`Condition_Is_Genuine_Existing_Page` → eventually `Create_Page_OneOff`, a path that expects `varTargetSectionPagesUrl` to already be populated, which it never is on a truly first-time capture). This produced the same `BadRequest`/empty-`sectionId` symptom as the earlier-logged bug, but from a completely different root cause.

## Why this matters more than a normal bug

**Flow Checker did not catch this.** Flow Checker's "Value is required" check (the one that's caught every corruption incident this week) only fires when a `SetVariable` action's `value` key is **missing entirely**. `OF05b` still had a `value` key present — it just held an empty string. That's structurally valid as far as Flow Checker is concerned, so it reported 0 errors while the flow was actually broken for every fresh one-off capture.

**This is very likely the same underlying corruption pattern seen all week (Pattern 6 / mass value-blanking), but manifesting in a form the team's primary detection method — Flow Checker — cannot see.** `OF05b` was one of the 26 actions restored during yesterday's two Express-mode corruption recoveries, set correctly at the time. It has since gone blank again, with **no corruption event, no error banner, and no Flow Checker warning** to signal it happened.

## Fix applied

David confirmed the value was missing and corrected `OF05b — Set varFinalPageDecision (OneOff)` back to `@outputs('OF03_—_Compose_PageDecision_OneOff')`, live, matching the same expression restored yesterday.

## Recommended next steps

1. **Spot-check `OF05a` and `OF05c`** (the two sibling actions either side of OF05b in the same one-off mapping-write group) for the same silent-blank pattern, since corruption this week has repeatedly affected clusters of related actions together, not single actions in isolation.
2. **Treat Flow Checker's "0 errors" as necessary but not sufficient** going forward — it did not protect against this. Where possible, spot-check actual `SetVariable` values via Peek Code/Code view after any period of inactivity or any settings change, not just after Designer edits.
3. **This is very strong, high-priority evidence for the Microsoft ticket** — arguably the single most important finding of the week: the corruption pattern can produce a *present-but-empty* value, not just a *missing* value, and the platform's own validation tooling does not detect the empty-value variant. This should be highlighted prominently and separately from the "missing value" incidents already documented.
4. Given this happened with no Designer edit, no publish, and no settings change in between yesterday's fix and today's discovery — genuinely at rest — this reinforces that periodic verification (not just post-edit verification) may be warranted for any flow known to have been affected by this pattern before.

---

**Status: root cause found (silent empty-value corruption, not a stale-mapping issue as first theorized) and fixed live. Original stale-mapping bug (`bug-2026-08-16-oneoff-stale-mapping-empty-sectionid.md`) remains separately valid and open for genuinely stale mappings — the two are different bugs that happened to produce the same visible symptom.**
