# Phase 2 Validation Plan

## Purpose

Phase 2 validates the actual Teams-OneNote-Meeting-Capture solution using the completed Build Coach framework.

The Build Coach programme is complete. Phase 2 uses that framework to validate the live user-facing Meeting Capture agent, Flow A, Flow B, OneNote output, and SharePoint mapping behaviour.

## Source of truth

GitHub remains the source of truth.

SharePoint remains the Copilot Studio Knowledge mirror.

The Build Coach is the guided operating framework.

## Scope

Phase 2 validates:

- user-facing Meeting Capture agent
- Flow A — meeting resolution
- Meeting Capture topic routing
- Flow B — OneNote and SharePoint orchestration
- UJ1–UJ5
- evidence capture
- controlled amendment process if failures occur

## Validation sequence

Validate one user journey at a time:

1. UJ1 — one-off single match
2. UJ2 — multiple match selection
3. UJ3 — recurring meeting with existing mapping
4. UJ4 — first-time recurring setup
5. UJ5 — no-match recovery

Do not proceed to the next user journey until the current journey has passed or the failure has been captured and handled through the controlled amendment process.

## Failure rule

If any validation fails:

1. Stop.
2. Capture evidence.
3. Identify the failure layer.
4. Identify the failure type.
5. Diagnose root cause.
6. Define one controlled change.
7. Update GitHub if baseline-relevant.
8. Mirror to SharePoint if required.
9. Refresh Knowledge if required.
10. Re-test the affected journey.

No ad hoc patching is permitted.
