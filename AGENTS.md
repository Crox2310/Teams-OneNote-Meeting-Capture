# Agent Instructions

This repository is intended to support AI-assisted development of the Teams-OneNote-Meeting-Capture project.

When working in this repository:

- Treat documentation in `docs/` as the source of truth.
- Respect the distinction between physical Agent Flows and user journeys.
- Do not merge Flow A and Flow B responsibilities.
- Do not add OneNote or SharePoint actions to Flow A.
- Protect the Flow B OneNote `/pages` normalisation rule.
- Keep build instructions precise enough for manual Copilot Studio / Agent Flow construction.
- When reviewing a proposed flow, simulate expressions, null handling, dynamic content, and type conversion risks.
