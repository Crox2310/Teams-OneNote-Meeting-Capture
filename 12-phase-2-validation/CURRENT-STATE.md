# CURRENT STATE — 22 Sep 2026

**Phase:** live, in field testing (since 20 Sep). The build is feature-complete; new work goes through `BACKLOG.md`.

This file replaces the previous CURRENT-STATE, which is still available in git history.

---

## What's live

| Component | State |
|---|---|
| Topic (Copilot Studio) | Published. Calls Flow A (C2) and Flow B (C10) directly. |
| Flow A — Resolve Meeting Selection | Published. EndTime taken as UTC from `end` (timezone fix, 21 Sep). |
| Flow B — Resolve OneNote Section | Published. Recurring pages go to per-series sections; one-off pages go to the fixed One-Off Meetings section. The one-off mapping row now includes OccurrenceDate, JoinUrl and EndTime (21 Sep). |
| Flow C — Chat Capture | Published. Appends the Copilot recap and the chat; supports one-offs through MeetingId (21 Sep). |
| Flow D — Auto Scheduler | Published. Polls every 10 minutes and handles recurring and one-off rows (FD04, FD04b, FD04c). |

Confirmed working end to end:

- recurring set-up and fill (STDA, row 364, 18 Sep)
- recap rendering in OneNote (16 Sep)
- one-off set-up (7 Sep, then re-routed to the fixed section on 18 Sep)

---

## Field testing

Started 20 Sep. Log findings in the **Field-testing intake** table in `BACKLOG.md`.

Watch in particular:

- **BL-01** — Flow D's FD06d skipped despite the True branch (22 Sep)
- **BL-02 / BL-03** — one-off JoinUrl and the one-off fill
- **BL-04** — Flow D offset value

---

## Top-level documents

- `ARCHITECTURE.md` — design, data model, decisions, constraints
- `USER-GUIDE.md` — using the agent
- `RUNBOOK.md` — checks, recovery, troubleshooting
- `BACKLOG.md` — open items and field-testing intake

## Key working files (this folder)

| File | Use |
|---|---|
| `known-good-values-master-reference.md` | Flow B values for recovery |
| `known-good-values-flow-a-reference.md` | Flow A values for recovery |
| `known-good-values-addendum-2026-09-18.md` | Flow A/B/Topic changes from 18 Sep, not yet merged into the references above (BL-17) |
| `flow-reference-2026-09-19-flow-c-published.md` | Flow C snapshot (the 21 Sep one-off changes aren't in it yet) |
| `flow-reference-2026-09-19-flow-d-published.md` | Flow D snapshot (FD04b and FD04c aren't in it yet) |
| `analysis-2026-09-19-end-to-end-review.md` | Findings F1–F10, the source of several backlog items |
| `microsoft-discussion-brief-corruption-bug.md` | Platform corruption evidence |
| `amendment-log.md` | Dated change log |

The latest handover is `handover-2026-09-20-tools-broken.md`. The tool links it describes were repaired before the 21 Sep work.
