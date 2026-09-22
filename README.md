# Teams → OneNote Meeting Capture

A Copilot Studio agent and four Power Automate flows that give every Teams meeting a OneNote page, and fill that page after the meeting with the Teams Copilot recap and the meeting chat.

**Status (22 Sep 2026):** live and in field testing. Known issues and improvements are in [`BACKLOG.md`](BACKLOG.md).

## What it does

1. **Set-up.** In Teams, you ask the agent to capture a meeting. It lists that day's meetings from your calendar; you can move to the previous or next day or jump to a date, then pick one. The agent creates a templated OneNote page in the right section and replies with a link to it.
   - Recurring meetings get one section per series and one page per occurrence.
   - One-off meetings go into a single One-Off Meetings section.
2. **Fill.** About 30 minutes after the meeting ends, a scheduler adds the Teams Copilot recap (meeting notes and action items, when one exists) and the meeting chat to the page. There is nothing to trigger.

## Documents

| Read this | For |
|---|---|
| [`ARCHITECTURE.md`](ARCHITECTURE.md) | How the components fit together, the data model, design decisions and constraints |
| [`USER-GUIDE.md`](USER-GUIDE.md) | Using the agent day to day |
| [`RUNBOOK.md`](RUNBOOK.md) | Checks, recovery and troubleshooting |
| [`BACKLOG.md`](BACKLOG.md) | Open bugs, items to verify, improvements, and field-testing intake |
| [`12-phase-2-validation/CURRENT-STATE.md`](12-phase-2-validation/CURRENT-STATE.md) | Current status and index of working files |

## Components

| Component | Name |
|---|---|
| Agent | Copilot Studio — Meeting Capture Topic |
| Flow A | PA - Meeting Capture - A - Resolve Meeting Selection |
| Flow B | PA - Meeting Capture - B - Resolve OneNote Section |
| Flow C | PA - Meeting Capture - C - Chat Capture |
| Flow D | PA - Meeting Capture - D - Auto Scheduler |
| Mapping list | SharePoint `RecurringMeetingSectionMap` |
| Notebook | OneNote `Meeting Notes` |

## Repository layout

Folders `00-` to `11-` hold the original design baseline (June 2026) and are kept for history; the system has moved on from them in places, and `ARCHITECTURE.md` takes precedence. `12-phase-2-validation/` is the working folder for everything since: dated session and handover notes, flow snapshots, and the known-good-values references used for recovery.
