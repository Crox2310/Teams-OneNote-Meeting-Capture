# Baseline Register

## Current trusted baseline

Baseline name:

Teams-OneNote-Meeting-Capture v1.0.0 — Validated Build Coach Baseline

Status:

Trusted v1 build and operational readiness baseline.

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
- Power Automate flows must not drift from the documented baseline.
- SharePoint Knowledge mirror must be updated only from approved GitHub baseline files.
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

## Build Coach programme completion

Status:

Complete

Confirmed:

- Build Coach 01–06 are complete.
- Stages 1–9 are complete.
- Final Build Coach 06 test session passed.
- Stage 9 Step 7 — Run production readiness review passed.
- MSG07A — Production readiness review completed appeared and the topic stopped correctly.
- The planned Build Coach programme is complete.
- Future work must be handled through controlled amendment or versioning.

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
- Direct Core Baseline Files are available as a high-signal knowledge source.
- The Build Coach agent can answer project-specific baseline questions from the baseline files.

Notes:

The Build Coach agent has been updated to prioritise the SharePoint knowledge source named:

Teams-OneNote-Meeting-Capture v1.0.0 Baseline

For baseline questions, the agent should use:

- 00-START-HERE-v1-baseline.md first
- baseline-register.md for current baseline state
- amendment-log.md for controlled amendment process
- build-coach-index.md for Build Coach scope

The agent should not rely on general OneNote, Teams, Microsoft support, or web content for project-baseline questions unless the maker explicitly asks for general product help.

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

- Ownership model has been reviewed.
- Connector account and permission model has been reviewed.
- Publishing and versioning process has been reviewed.
- Monitoring and diagnostics approach has been reviewed.
- Support and failure-handling process has been reviewed.
- Controlled amendment operating model has been reviewed.
- Production readiness review has been completed.

## Production readiness outcome

Readiness decision:

GO

Review notes:

The Build Coach programme has completed its planned build, validation, baselining, knowledge grounding, operationalisation, and production readiness coverage.

Confirmed:

- GitHub source of truth is updated.
- SharePoint Knowledge mirror is updated.
- Core Baseline Files are available as direct knowledge.
- Build Coach Knowledge grounding is validated.
- Build Coach 01–06 are complete.
- Stages 1–9 are complete.
- Final Build Coach 06 test passed.
- Stage 9 production readiness review completed.
- Controlled amendment operating model is defined.
- Future changes must use controlled amendment or versioning.

Conditions:

None currently recorded.

## Final completion note

A final completion note records the completion of the Build Coach programme.

Location:

11-baseline/final-completion-note.md

The final completion note confirms:

- Build Coach 01–06 complete
- Stages 1–9 complete
- GitHub source of truth aligned
- SharePoint Knowledge mirror aligned
- Build Coach Knowledge grounding validated
- operational readiness completed
- future changes governed through controlled amendment or versioning

## Future controlled changes

After Stage 9, future work should be handled through:

- controlled amendment
- v1.0.x operational or documentation update
- v1.1.0 minor backward-compatible enhancement
- v2.0.0 major design or architecture change

No further uncontrolled build stages should be added to the v1 baseline.

## Current operating model

The Build Coach is no longer in build mode.

The Build Coach is now the controlled operating framework for:

- future rebuilds
- future validation
- controlled amendments
- support triage
- connector learning capture
- versioned enhancements
- production readiness checks

Any future defect, connector learning, design correction, build correction, test correction, or operational readiness change must be handled through the controlled amendment process recorded in:

11-baseline/amendment-log.md
