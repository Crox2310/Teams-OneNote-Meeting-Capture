# Handover — 16 August 2026 (live, at work) — Third corruption event today; `OF05a/b/c` group did NOT recur, everything else did

## START HERE

This continues directly from `bug-2026-08-16-of05b-silent-empty-value-flowchecker-blind.md`, where the `OF05a`/`OF05b`/`OF05c` cluster was found silently blanked (present-but-empty values, invisible to Flow Checker) and fixed live at work. **Shortly after that fix, Flow Checker reported 23 errors** — the standard "missing value key" corruption signature (distinct from the empty-value variant just fixed), affecting a large but different set of actions.

**This is the third corruption event in roughly 24 hours** (two yesterday, both triggered by the Express-mode toggle; this one with no settings change or Designer edit in between — the flow was simply being used normally, live, in production).

---

## What happened

After confirming the `OF05a/b/c` cluster fixed and Flow Checker clean, David continued live testing at work. Shortly after, Flow Checker jumped to **23 errors** — the standard "missing `value` key" signature seen throughout this week, covering the recurring/one-off mapping-write and page-creation `SetVariable` groups (the same category as yesterday's 26-error incidents, just 23 this time — a slightly different count/membership).

**Notably, the `OF05a`/`OF05b`/`OF05c` group — just fixed moments earlier in the empty-value incident — was NOT among the 23 corrupted actions this time.** This is useful evidence: the two corruption variants (present-but-empty vs. missing-key) do not appear to always hit the same action set together, and fixing one group does not appear to make it more or less likely for a different group to be hit shortly after by the other variant.

## Recovery

Same established discipline: all 23 values re-entered one at a time, saved individually, Flow Checker checked after each. Full list matches the standard reference values used in all prior recoveries this week (see `handover-2026-08-16-express-mode-corruption-trigger-recovered.md` for the canonical source of these expressions). **Confirmed: Flow Checker 0 errors after full recovery.**

## Why this occurrence is significant

- **No settings change, no Designer edit, and no publish event preceded this incident** — it happened during ordinary live use (testing UJ1/UJ3/UJ4 in a real Teams conversation at work), the same "at rest, no trigger" pattern as the empty-value `OF05a/b/c` discovery earlier the same session.
- **Combined with the empty-value finding earlier today, this means at least two structurally different corruption events occurred within roughly 20 minutes of each other, during ordinary use, with no apparent user action triggering either.** This further weakens any theory that careful editing discipline alone can prevent these incidents — both today's occurrences happened with the flow simply being used as intended.
- Three total corruption/recovery cycles inside 24 hours (two 15/16 August tied to Express mode, this one untied to any known trigger) is a strong, mounting pattern that should be the headline of the Microsoft ticket, not a footnote.

## Recommended next steps

1. **Add this third occurrence to the Microsoft ticket** alongside the Express-mode findings and the empty-value discovery — specifically noting that this one occurred with no identifiable trigger at all, which is the most concerning variant so far.
2. Given the frequency today, **consider a brief Flow-Checker check as a standing habit before and after every live-testing block** for the remainder of this investigation, not just after edits — cheap, fast, and would have caught this immediately rather than via a failed live test.
3. Continue tracking whether `OF05a/b/c` specifically remains stable now that it's been fixed twice today, versus other action groups continuing to be hit — this pattern (which group, which variant, how often) is exactly the kind of detail Microsoft will likely want for their own investigation.
4. All other items from earlier remain open and unaffected by this: the `bug-2026-08-16-oneoff-stale-mapping-empty-sectionid.md` design gap (still a real, separate issue, distinct from both of today's corruption events), the recurring title-set race condition, and the broader UJ regression pass David is mid-way through at work.

---

**Status: third corruption event of the 24-hour window recovered cleanly (23/23 values restored, Flow Checker 0 errors). Occurred with no identifiable trigger, during ordinary live use. This, combined with the earlier empty-value discovery in the same session, represents the strongest evidence yet that this is a platform-level issue outside the team's control, not something preventable through editing discipline alone.**
