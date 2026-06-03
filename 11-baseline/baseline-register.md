# Baseline Register

## Current trusted baseline

Baseline name:

Teams-OneNote-Meeting-Capture v1.0.0 — Validated Build Coach Baseline

Status:

Trusted v1 build baseline.

## Included scope

This baseline includes:

- Final Claude-amended design baseline
- Shared Journey Contract vFinal
- Outlook Meeting Data Capture Profile V1
- User Journeys UJ1–UJ5
- Build Coach 01 — Setup and Flow A Foundation
- Build Coach 02 — Meeting Capture Topic Build
- Build Coach 03 — Flow B Connector Validation
- Build Coach 04 — Flow B Build
- Build Coach 05 — End-to-End Testing and Controlled Amendment Process

## Validated stages

- Stage 1 — Environment and SharePoint setup
- Stage 2 — Flow A shell and Outlook connector debug
- Stage 3 — Outlook schema inspection
- Stage 4 — Flow A mapping and validation
- Stage 5 — Meeting Capture topic routing
- Stage 6 — Flow B connector validation
- Stage 7 — Flow B build
- Stage 8 — End-to-end testing and controlled amendment process

## Baseline principles

- GitHub is the source of truth.
- SharePoint is the Copilot Knowledge mirror.
- Copilot Studio implementation must not drift from GitHub.
- Build failures must be diagnosed before correction.
- Fixes must be documented before implementation.
- No ad hoc patching is permitted.

## Current Copilot Studio topics

The following Build Coach topics form the validated v1 baseline:

- Build Coach 01 — Setup and Flow A Foundation
- Build Coach 02 — Meeting Capture Topic Build
- Build Coach 03 — Flow B Connector Validation
- Build Coach 04 — Flow B Build
- Build Coach 05 — End-to-End Testing and Controlled Amendment Process

## Knowledge grounding validation

Status:

Passed

Validated with:

- 00-START-HERE-v1-baseline.md
- amendment-log.md

Confirmed behaviour:

- Core baseline files are retrievable.
- Baseline concepts are summarised correctly.
- Controlled amendment process is recognised.
- SharePoint Knowledge mirror is usable by the Build Coach.

Notes:

The Build Coach agent has been updated to prioritise the SharePoint knowledge source named:

Teams-OneNote-Meeting-Capture v1.0.0 Baseline

For baseline questions, the agent should use 00-START-HERE-v1-baseline.md first and should not rely on general OneNote, Teams, Microsoft support, or web content unless the maker explicitly asks for general product help.

## Next controlled phase

Operationalisation and production readiness should be handled as a future controlled baseline extension.
