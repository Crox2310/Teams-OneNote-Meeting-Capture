# Teams-OneNote-Meeting-Capture

## Purpose

Teams-OneNote-Meeting-Capture is a Copilot Studio and Agent Flow project to capture meeting notes from Microsoft Teams / Outlook meetings into structured OneNote pages.

The solution is designed around a simple operating model:

```text
Copilot Studio Topic = user journey and orchestration
Agent Flow A = Outlook meeting lookup
Agent Flow B = OneNote capture and recurring meeting mapping
OneNote = clean human-readable meeting record
SharePoint = recurring meeting mapping store
```

## Current baseline

The current agreed build baseline is:

```text
Agent Flow A v3.1 — Resolve Meeting Selection — Corrected Outlook Lookup Baseline
```

Flow A should be built manually as:

```text
PA - Resolve Meeting Selection - v1 Clean Build
```

Flow A uses only the Office 365 Outlook connector and the action **Get calendar view of events**.

## Key principles

- Build from a clean baseline.
- Do not patch endlessly.
- Rename every action immediately.
- Do not rename actions after expressions reference them.
- All Agent Flow outputs to the Topic are strings.
- Flow A does not touch OneNote or SharePoint.
- Flow B owns OneNote, SharePoint mapping, and the `/pages` safeguard.
- Attendees are deferred to v2 enrichment.
- Debug outputs are testing only.
- User-facing OneNote pages must remain clean and human-readable.

## Repo navigation

| Area | Location |
|---|---|
| Project overview | `docs/00-project/` |
| Architecture | `docs/01-architecture/` |
| User journeys | `docs/02-user-journeys/` |
| Agent flows | `docs/03-agent-flows/` |
| Topic design | `docs/04-topic-design/` |
| Test packs | `docs/05-testing/` |
| Design decisions | `docs/06-decisions/` |
| Claude stress-test prompts | `docs/07-claude-review-prompts/` |
| Build checklists | `docs/08-build-checklists/` |
| Reference notes | `docs/09-reference/` |
| GitHub Copilot instructions | `.github/copilot-instructions.md` |

## Recommended next build action

Build and validate Agent Flow A v3.1 manually before wiring it into the Copilot Studio topic.
