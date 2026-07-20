# Baseline Register

## Current trusted baseline

Baseline name:

Teams-OneNote-Meeting-Capture v1.0.1 — Validated Build Coach Baseline

Status:

Trusted v1 build and operational readiness baseline, corrected 2026-07-20.

## Baseline correction — 2026-07-20

The original v1.0.0 baseline declared a production-readiness decision of "GO" following the Build Coach programme's Stage 8 (end-to-end testing) and Stage 9 (operationalisation and production readiness) completion. That declaration was made before the following was known:

- `Condition_IsRecurring` in Flow B never evaluated True for any meeting, for the entire project history, due to a string/boolean type-coercion defect in the Condition builder (AMEND-2026-07-19-001).
- As a direct consequence, UJ3 (recurring, existing mapping) and UJ4 (recurring, first-time setup) had never been correctly exercised. Every prior "successful" recurring-meeting test had silently fallen through to the one-off code path instead.
- UJ5 (no-match recovery) had never been tested at all.
- Day-navigation (P/N) did not actually shift the query date, due to a trigger schema field-name mismatch (AMEND-2026-07-18-001).

As of 2026-07-19/20, all nine amendments logged in `amendment-log.md` under "Logged amendments — backfilled 2026-07-20" have been applied and retested, and all five original user journeys (UJ1–UJ5) have been independently confirmed working via live testing with Activity-trace evidence, for the first time in the project's history. See `12-phase-2-validation/` for the individual UJ1–UJ5 validation records and `2026-07-20-date-jump-feature-and-uj-validation.md` for the combined summary.

This baseline is therefore corrected from v1.0.0 to v1.0.1 to reflect a genuine, verified state rather than the originally-declared one. The production readiness decision below is re-confirmed on that corrected basis, with open items carried forward explicitly rather than implied as resolved.

## Included scope

This baseline includes:

- Final Claude-amended design baseline
- Shared Journey Contract vFinal
- Outlook Meeting Data Capture Profile V1
- User Journeys UJ1–UJ5 — now genuinely validated, see correction above
- Day-navigation (P/N) and date-jump navigation — an approved design enhancement beyond the original UJ1–UJ5 scope, layered on top of UJ2's day-query mechanism. Superseded the original UJ5 recovery menu ("search again with different wording" / "Stop"); a follow-up build item to add a reword/retry option and explicit Stop alongside date-jump is agreed but not yet built.
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
- Stage 8 — End-to-end testing and controlled amendment process — re-confirmed 2026-07-20 on corrected basis (see Baseline correction above); the controlled amendment process itself was not actually used for the fixes that got UJ1–UJ5 working, and has been backfilled retroactively into `amendment-log.md`
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

Complete, with the controlled amendment process backfilled 2026-07-20 (see Baseline correction).

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

Pending action: rename or version this SharePoint knowledge source to v1.0.1 once this corrected register and the backfilled amendment log are mirrored, so the Build Coach's own knowledge grounding does not reference a superseded baseline name.

For baseline questions, the agent should use:

- 00-START-HERE-v1-baseline.md first
- baseline-register.md for current baseline state
- amendment-log.md for controlled amendment process
- build-coach-index.md for Build Coach scope

The agent should not rely on general OneNote, Teams, Microsoft support, or web content for project-baseline questions unless the maker explicitly asks for general product help.

## Operational readiness validation

Status:

Completed, with two items carried forward as open rather than resolved (see below).

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
- Connector account and permission model has been reviewed. Open item: connections have repeatedly gone stale (`FlowActionBadGateway`) after publishing throughout 2026-07-18 to 2026-07-20, requiring manual reconnection each time. This has been worked around operationally but not resolved architecturally — worth a deliberate decision on whether connections should move off a personal OAuth session onto a more durable account.
- Publishing and versioning process has been reviewed.
- Monitoring and diagnostics approach has been reviewed. Open item: no active failure alerting has been confirmed as configured; the `Set_varOutStatus` regression (AMEND-2026-07-19-005) was only caught because of active manual testing, not any monitoring signal.
- Support and failure-handling process has been reviewed.
- Controlled amendment operating model has been reviewed. Open item: the process was not actually followed for AMEND-2026-07-18-001 through AMEND-2026-07-20-001 at the time the work was done; all nine have been backfilled retroactively as of 2026-07-20.
- Production readiness review has been completed.

## Production readiness outcome

Readiness decision:

GO, re-confirmed 2026-07-20 on the corrected v1.0.1 basis.

Review notes:

The Build Coach programme has completed its planned build, validation, baselining, knowledge grounding, operationalisation, and production readiness coverage. This baseline register was corrected 2026-07-20 to reflect that the original v1.0.0 "GO" decision predated the discovery and fix of significant defects in the recurring-meeting path (see Baseline correction above). UJ1–UJ5 are now genuinely validated for the first time.

Confirmed:

- GitHub source of truth is updated.
- SharePoint Knowledge mirror update pending — see Knowledge grounding validation notes above.
- Core Baseline Files are available as direct knowledge.
- Build Coach Knowledge grounding is validated, pending the v1.0.1 rename noted above.
- Build Coach 01–06 are complete.
- Stages 1–9 are complete.
- Final Build Coach 06 test passed.
- Stage 9 production readiness review completed.
- Controlled amendment operating model is defined and, as of 2026-07-20, actually in use (backfilled).
- Future changes must use controlled amendment or versioning.

Conditions:

- Mirror this corrected register and the backfilled `amendment-log.md` to SharePoint Knowledge, and rename the SharePoint knowledge source from "v1.0.0 Baseline" to "v1.0.1 Baseline."
- Take a Power Platform solution export of Flow A and Flow B as a re-importable backup of the current working v1.0.1 state, and commit the current Meeting Capture Topic YAML to GitHub as the canonical version (not just referenced in dated session docs).
- Screenshot the full Flow A, Flow B, and Topic configurations for visual reference, once the above exports are taken.
- Resolve or formally accept the two open operational items above (connection durability, failure alerting) as known risks.
- Document the `RecurringMeetingSectionMap` SharePoint list's internal column names explicitly, given this project has twice been affected by internal-vs-display field name mismatches (AMEND-2026-07-18-001 and the OneNote section-naming defect).
- Update `02-user-journeys/user-journey-5-no-match-recovery.md` once the agreed reword/retry + Stop addition to date-jump navigation is built, so the on-paper spec matches shipped behaviour.
- The `OutStatus` differentiation gap (single hardcoded value vs. the six-state contract in `shared-journey-contract-vfinal.md`) remains open and is the prerequisite for several other tracked gaps (UJ3 stale/duplicate-row handling, UJ4 section-choice and blank-`SeriesMasterId` fallback, `SectionRetryCount`). See `2026-07-20-gap-analysis-original-brief-vs-current-build.md` for the full breakdown and recommended build order.

## Final completion note

A final completion note records the completion of the Build Coach programme.

Location:

11-baseline/final-completion-note.md

The final completion note confirms:

- Build Coach 01–06 complete
- Stages 1–9 complete
- GitHub source of truth aligned
- SharePoint Knowledge mirror aligned — pending v1.0.1 mirror update, see Conditions above
- Build Coach Knowledge grounding validated
- operational readiness completed, with two open items carried forward (see above)
- future changes governed through controlled amendment or versioning

## Future controlled changes

After Stage 9, future work should be handled through:

- controlled amendment
- v1.0.x operational or documentation update
- v1.1.0 minor backward-compatible enhancement
- v2.0.0 major design or architecture change

No further uncontrolled build stages should be added to the v1 baseline. The nine amendments backfilled 2026-07-20 are the last changes permitted to be logged retroactively — from this point forward, the amendment process in `amendment-log.md` must be followed at the time a defect or design correction is found, not after the fact.

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
