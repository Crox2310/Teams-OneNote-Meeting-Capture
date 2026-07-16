# Process — Maintaining the Living Audit

This document governs how `living-audit.md` is kept up to date. It does not contain the audit itself — see `living-audit.md` for the current per-action expression catalogue.

## Core principle

`living-audit.md` is **current ground truth, not a session log**. It should always reflect the actual state of Flow A and Flow B's expressions as they exist in Designer right now. It is not a diary of what happened in a given session — that's what the dated `handover-*.md` files are for.

If a handover and the living audit ever disagree about the status of a specific action, the living audit is wrong and needs fixing immediately — don't let the discrepancy stand.

## When to update

Update `living-audit.md` **the moment an expression changes in Designer** — not at the end of the session, not "later." Specifically:

- Immediately after fixing an expression via Code view or the expression editor, before moving to the next action.
- Before closing out the session's handover note. The handover should never be the only place a fix is recorded — if it's not in the living audit too, the fix will get re-discovered and re-investigated in a future session.
- Before marking any UJ (user journey) validation as passed, confirm every action it touched is reflected accurately in the audit.

## Status key

Use these four states consistently — do not invent new ones:

- 🔴 confirmed bug, not fixed
- 🟡 suspect/unconfirmed — flagged as a possible issue but not yet verified either way
- 🟢 confirmed fixed and tested
- ⚪ confirmed clean (no issue)

An action should never be left unmarked. If you haven't looked at it yet, it isn't in scope for the current audit pass — don't guess a status.

## What counts as "confirmed"

Before marking anything 🟢 or ⚪, verify it through at least one of:

- The expression is visible and correct in the Designer expression editor (chip or tooltip), **or**
- The raw JSON is visible and correct in Code view, **or**
- A live test run's actual input/output values were inspected in Activity → Run results and matched expectation.

A status based on memory of "I think I fixed that already" is not confirmed — re-check it in Designer or Code view before changing its mark. This is exactly the kind of drift that produced the P/N navigation bug: multiple fixes were believed to be live but hadn't actually propagated to production.

## Bug pattern reference

When flagging or fixing an issue, tag it against this list where it applies — this makes it easier to spot when the same root cause is recurring across multiple actions:

1. String-wrapped booleans / missing `empty()` guards
2. Blank or literal-`''` values
3. Wrong/mismatched field names
4. Missing `@` expression prefix
5. Wrong-array indexing — indexing an unfiltered array using an index/count derived from a filtered array (or vice versa)
6. Parameter/sentinel mismatch — a value passed by the Topic that the Flow's condition logic doesn't recognize (e.g. a direction letter landing in a field only built to handle numbers or a fixed sentinel string)

Extend this list as new patterns are found rather than describing a new bug from scratch each time — a shared taxonomy makes cross-flow audits faster.

## Format conventions

Follow the structure already established in `living-audit.md`:

- Group by flow, then by the order actions actually run in (trigger → variable setup → branching → response).
- Reference actions by their exact Designer name (e.g. `FA16 Compose SelectedIndex`), bolded, followed by the status icon.
- Where a fix was applied, show the corrected expression in a code block and a one-line note on what was wrong before.
- Keep an "Open items / not yet covered by this audit" section at the end for anything flagged but not yet resolved — this is the actual backlog, not a place fixes go to be forgotten.

## Ownership

Whoever makes a change in Designer is responsible for updating the audit before ending the session. If a session runs out of time before the audit can be updated, say so explicitly in that session's handover note (e.g. "Living audit not yet updated for FA15/FA16 changes made this session") so the next session knows not to trust the audit blindly for those actions.
