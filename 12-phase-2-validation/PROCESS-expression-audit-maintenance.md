# Process Note: The Living Audit — Maintenance Instructions for Future Sessions

## What this document is

This is a standing process instruction, not a dated session handover. Read it at the start of any session involving `living-audit.md` or `living-audit-topic.md` in this repo, alongside the most recent dated handover note.

There are two companion living documents this process note governs:

- **`living-audit.md`** — the per-action expression catalogue for Flow A and Flow B. Current ground truth for every reviewed action's expression, organized by flow and by action, with a status flag per entry (🔴 confirmed bug · 🟡 suspect · 🟢 fixed and tested · ⚪ confirmed clean).
- **`living-audit-topic.md`** — broader system documentation for the Copilot Studio Topic orchestration layer (which flow gets called when, with what inputs, branch structure, response messages, live-test status per user journey). Companion to the expression catalogue, not a replacement for it — the Topic document answers "what is this supposed to do and has it been verified," the expression catalogue answers "is this specific piece of code correct."

This document (`PROCESS-expression-audit-maintenance.md`) only describes how to maintain both — it has no per-action or per-node content itself.

## Background

Starting 2026-06-20, David and Claude began a full Code View audit of every action's expression across Flow A (`PA - Resolve Meeting Selection - v1 Clean Build`) and Flow B (`PA - Resolve OneNote Meeting Section - v2 Clean Build`). This was prompted by a recurring bug pattern discovered across multiple sessions: type mismatches and silent failures hiding in expressions that look fine in Designer's collapsed view but are visibly wrong in Code View. Three confirmed recurring patterns:

1. **String-wrapped booleans** — comparisons like `"@true"` (a string literal) being checked against an evaluated boolean, or an entire expression wrapped in quotes as `"@greater(...)"` rather than evaluated natively. Also includes the related "missing `empty()` guard" variant: `coalesce(var, '0')` looks safe but only catches `null`, not `''` — so `int(coalesce(var, '0'))` still throws if `var` is an empty string. Confirmed in `Condition Should Create Page` (fixed, earlier session) and `Condition_Should_Write_Mapping` (root-caused 2026-06-20, fix drafted, not yet applied).
2. **Blank/literal-`''` values** — `Set variable` or `Compose` actions with an empty or literal-string `''` Value/Inputs field instead of a real expression or `string('')`. Confirmed in multiple actions across both flows on 2026-06-14 and again on 2026-06-20 (`FA33A`, `FA34A` — same actions, second occurrence; ~15-20 further instances catalogued in Flow B on 2026-06-20, see `living-audit.md`).
3. **Wrong/mismatched field names** — expressions reading a property name that doesn't exist on the source object (wrong casing, made-up field name, or a generic trigger field name like `text`/`text_1` being misread as a named field), silently returning null/false instead of erroring. Confirmed in `FA28A_Compose_OutIsRecurring`, `FA28B_Compose_OutSeriesMasterId` (both fixed 2026-06-20), and `Condition_IsRecurring`'s trigger-key bug in Flow B (root-caused 2026-06-20, fix not yet applied).

On 2026-06-21, scope was extended upward to the Topic orchestration layer via `living-audit-topic.md`, after recognizing that several of the above bugs originate at the seam between the Topic and the flows (e.g. the trigger field mapping issue) — a seam that the expression catalogue alone, being flow-internal, can't fully document.

The audit's purpose is to catch all remaining instances of these patterns in one pass, rather than continuing to discover them one at a time through live debugging — which is expensive in both time and live-system risk (see the connection-incident handover from 2026-06-20 evening).

## The core instruction for future sessions

**Both `living-audit.md` and `living-audit-topic.md` are snapshots, not a live sync.** Each is only accurate as of the last time it was explicitly updated. Any session in which a connector/action/node is added, amended, or deleted in Designer or in Copilot Studio's authoring canvas — whether as a bug fix, a deliberate enhancement, a new step, or a removed one — MUST end with the relevant document being updated to reflect the change, before the session's handover note is finalized.

This applies even to small or seemingly trivial edits, and applies equally to additions and deletions, not just amendments. The whole value of these documents is that they can be trusted as accurate; a single un-logged drift makes every future read of them less reliable.

**Which document to update:** flow-internal expression/action changes (Power Automate, Code View) go in `living-audit.md`. Topic-level changes (Copilot Studio authoring canvas — node structure, branch logic, trigger configuration, response message text, Topic-to-flow input/output mapping) go in `living-audit-topic.md`. Section 4 of `living-audit-topic.md` (the Topic-to-flow contract table) is the one place that deliberately straddles both — keep it in sync with `living-audit.md`'s trigger and Respond-to-agent sections when either changes.

## What "updating the Living Audit" means in practice

When a connector/action/node changes during a session:

1. **Find or create its entry.** Both documents are organized top-to-bottom in execution order — `living-audit.md` by flow then by logical section then by action name as it appears in Designer; `living-audit-topic.md` by orchestration stage. If new, add the entry in the right place rather than appending to the end.
2. **Amended:** replace the old expression/description with the new one. Update the status flag (e.g. 🔴 → 🟢 once fixed and tested, or ⚪ → 🟡 if a new edit introduces a question mark worth flagging; `living-audit-topic.md` also uses 🔵 for "designed but not yet live-tested").
3. **New:** add a full entry — name, expression/parameters/description, status flag, and a one-line note on what it's for if not obvious from the name.
4. **Deleted:** remove its entry entirely rather than leaving a stale reference. If the deletion is itself noteworthy, a one-line note in the surrounding text is enough — don't leave a "this used to exist" tombstone entry, since that defeats the point of the doc being current ground truth.
5. **Add a one-line dated note** in the entry if the change is non-obvious (e.g. "2026-06-21: changed to use seriesMasterId per FA28A pattern fix").
6. **Update the "Last updated" line and "Coverage"/"Status" lines** at the top of whichever document changed.

Claude should do this proactively near the end of any session where connectors or Topic structure changed — don't wait to be asked, but do confirm the wording with David before committing, the same way other handover docs are confirmed before upload (GitHub MCP write access has been unreliable for this repo across sessions — confirm whether it's working at the start of each session; if not, prepare the file via `bash_tool`/`create_file` for manual upload via the GitHub web UI, as has been the standing workaround. **Note (2026-06-21): manual uploads have previously resulted in this process note and `living-audit.md` swapping content/filenames — always verify the uploaded file's content matches its intended filename via `github:get_file_contents` after any manual upload, before assuming it landed correctly.**)

## Why this matters going forward

Without this discipline, these documents will silently go stale within a session or two, at which point they become actively misleading rather than just incomplete — worse than not having them at all, since future sessions (including different Claude instances, given memory does not persist between conversations) would trust outdated information as current. The dated handover notes capture *what happened in a session*; `living-audit.md` and `living-audit-topic.md` are meant to capture *current ground truth*, and only stay useful if treated as such.

## Pointer for future Claude instances

If you are picking up this project fresh: read this file first, then the most recent dated handover note, then `living-audit-topic.md` for the system-level picture, then `living-audit.md` for flow-level expression detail — in that order, broad to narrow. If either living document doesn't exist yet, that scope of the audit hasn't started — check the most recent handover for context on what would need covering first.
