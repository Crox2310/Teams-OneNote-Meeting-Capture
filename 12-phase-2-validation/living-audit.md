# Process Note: The Living Audit — Maintenance Instructions for Future Sessions

## What this document is

This is a standing process instruction, not a dated session handover. It should be read by Claude at the start of any session involving the Living Audit (`living-audit.md` in this repo), alongside the most recent dated handover note.

The Living Audit is the name for the ongoing, continuously-updated expression reference described below — as opposed to the dated handover notes, which are point-in-time session logs. The name is deliberate: this document is only useful as long as it stays "alive," i.e. accurate as of right now. The moment it stops being updated, it becomes misleading rather than just stale.

## Background

Starting 2026-06-20, David and Claude began a full Code View audit of every action's expression across Flow A (`PA - Resolve Meeting Selection - v1 Clean Build`) and Flow B (`PA - Resolve OneNote Meeting Section - v2 Clean Build`). This was prompted by a recurring bug pattern discovered across multiple sessions: type mismatches and silent failures hiding in expressions that look fine in Designer's collapsed view but are visibly wrong in Code View. Three confirmed recurring patterns:

1. **String-wrapped booleans** — comparisons like `"@true"` (a string literal) being checked against an evaluated boolean, or an entire expression wrapped in quotes as `"@greater(...)"` rather than evaluated natively. Confirmed in `Condition Should Create Page` (fixed, earlier session) and `Condition_Should_Write_Mapping` (found 2026-06-20, not yet fixed).
2. **Blank/literal-`''` values** — `Set variable` or `Compose` actions with an empty or literal-string `''` Value/Inputs field instead of a real expression or `string('')`. Confirmed in multiple actions across both flows on 2026-06-14 and again on 2026-06-20 (`FA33A`, `FA34A` — same actions, second occurrence).
3. **Wrong/mismatched field names** — expressions reading a property name that doesn't exist on the source object (wrong casing, made-up field name), silently returning null/false instead of erroring. Confirmed in `FA28A_Compose_OutIsRecurring` and `FA28B_Compose_OutSeriesMasterId` on 2026-06-20.

The Living Audit's purpose is to catch all remaining instances of these three patterns in one pass, rather than continuing to discover them one at a time through live debugging — which is expensive in both time and live-system risk (see the connection-incident handover from 2026-06-20 evening).

## The core instruction for future sessions

**The Living Audit is a snapshot, not a live sync.** It is only accurate as of the last time it was explicitly updated. Any session in which an action's expression is changed in Designer — whether as a bug fix, a deliberate enhancement, or any other edit — MUST end with the Living Audit being updated to reflect the new expression, before the session's handover note is finalized.

This applies even to small or seemingly trivial edits. The whole value of the Living Audit is that it can be trusted as accurate; a single un-logged drift makes every future read of it less reliable.

## What "updating the Living Audit" means in practice

When an expression changes during a session:
1. Identify the action's entry in the Living Audit (organized by flow, then by action name).
2. Replace the old expression text with the new one.
3. Update the action's flagged status if relevant (e.g. from "confirmed bug" to "fixed," or from "clean" to "suspect" if a new edit introduces a question mark).
4. Add a one-line dated note if the change is non-obvious (e.g. "2026-06-21: changed to use seriesMasterId per FA28A pattern fix").

Claude should do this proactively near the end of any session where expressions changed — don't wait to be asked, but do confirm with David before committing, the same way other handover docs are confirmed before upload (since GitHub MCP write access is broken for this repo and uploads are manual via the web UI).

## Why this matters going forward

Without this discipline, the Living Audit will silently go stale within a session or two, at which point it becomes actively misleading rather than just incomplete — worse than not having it at all, since future sessions (including different Claude instances, given memory does not persist between conversations) would trust outdated information. The dated handover notes capture *what happened in a session*; the Living Audit is meant to capture *current ground truth*, and only stays useful if it's treated as such.

## Pointer for future Claude instances

If you are picking up this project fresh: read this file first, then the most recent dated handover note, then `living-audit.md` itself. If `living-audit.md` doesn't exist yet, the Living Audit is still in progress — check the most recent handover for where it was left off.
