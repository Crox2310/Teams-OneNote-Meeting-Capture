# Baseline Register

## Current trusted baseline

Baseline name:

Teams-OneNote-Meeting-Capture v1.0.0 — Validated Build Coach Baseline

Status:

Trusted v1 build baseline with operational readiness extension completed.

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
- Build Coach 06 — Operationalisation and Production Readiness

## Validated stages

- Stage 1 — Environment and SharePoint setup
- Stage 2 — Flow A shell and Outlook connector debug
- Stage 3 — Outlook schema inspection
- Stage 4 — Flow A mapping and validation
- Stage 5 — Meeting Capture topic routing
- Stage 6 — Flow B connector validation
- Stage 7 — Flow B build
- Stage 8 — End-to-end testing and controlled amendment process
- Stage 9 — Operationalisation and production readiness

## Baseline principles

- GitHub is the source of truth.
- SharePoint is the Copilot Knowledge mirror.
- Copilot Studio implementation must not drift from GitHub.
- Build failures must be diagnosed before correction.
- Fixes must be documented before implementation.
- No ad hoc patching is permitted.
- Future changes must be handled through the controlled amendment process or formal versioning model.

## Current Copilot Studio topics

The following Build Coach topics form the validated v1 baseline:

- Build Coach 01 — Setup and Flow A Foundation
- Build Coach 02 — Meeting Capture Topic Build
- Build Coach 03 — Flow B Connector Validation
- Build Coach 04 — Flow B Build
- Build Coach 05 — End-to-End Testing and Controlled Amendment Process
- Build Coach 06 — Operationalisation and Production Readiness

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

## Operational readiness validation

Status:

Completed

Validated through:

- Stage 9 Step 1 — Confirm environments and ownership
- Stage 9 Step 2 — Confirm connector accounts and permissions
- Stage 9 Step 3 — Confirm publishing and versioning process
- Stage 9 Step 4 — Confirm monitoring and diagnostics approach
- Stage 9 Step 5 — Confirm support and failure-handling process
- Stage 9 Step 6 — Confirm controlled amendment operating model
- Stage 9 Step 7 — Run production readiness review

Confirmed behaviour:

- ownership model has been reviewed
- connector account and permission model has been reviewed
- publishing and versioning process has been reviewed
- monitoring and diagnostics approach has been reviewed
- support and failure-handling process has been reviewed
- controlled amendment operating model has been reviewed
- production readiness review has been completed

## Production readiness outcome

Readiness decision:

To be recorded as one of:

- GO
- GO WITH CONDITIONS
- NO-GO

Review notes:

Record final readiness notes here after Stage 9 Step 7 testing is complete.

## Future controlled changes

After Stage 9, future work should be handled through:

- controlled amendment
- v1.0.x operational or documentation update
- v1.1.0 minor enhancement
- v2.0.0 major design or architecture change

No further uncontrolled build stages should be added to the v1 baseline.
