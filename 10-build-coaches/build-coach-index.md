# Build Coach Index

## Purpose

This folder documents the guided Build Coach topics used to build the Teams-OneNote-Meeting-Capture solution in a controlled, step-by-step way.

The Build Coach topics are designed to prevent ad hoc patching, uncontrolled looping, connector drift, and undocumented fixes.

## Build Coach Topics

### Build Coach 01 — Setup and Flow A Foundation

Status: Complete

Scope:

- Stage 1 — Environment and SharePoint setup
- Stage 2 — Flow A shell and Outlook connector debug
- Stage 3 — Outlook schema inspection
- Stage 4 — Flow A mapping and validation

Outcome:

Flow A foundation is built, validated, and ready for downstream topic routing.

---

### Build Coach 02 — Meeting Capture Topic Build

Status: Complete

Scope:

- Stage 5 — Copilot Studio Meeting Capture topic skeleton
- Topic variables
- Initial Flow A call
- SINGLE_MATCH routing
- MULTIPLE_MATCHES routing
- NO_MATCH routing
- ERROR routing

Outcome:

The user-facing Meeting Capture topic has deterministic Flow A routing coverage.

---

### Build Coach 03 — Flow B Connector Validation

Status: Complete

Scope:

- Stage 6 — Flow B connector validation
- SharePoint access
- SharePoint internal column names
- SeriesMasterId OData filter
- OneNote notebook access
- OneNote Get sections output
- /pages URL normalisation
- OneNote page creation output

Outcome:

SharePoint and OneNote connector behaviours are validated before Flow B build.

---

### Build Coach 04 — Flow B Build

Status: Complete

Scope:

- Stage 7 — Flow B build
- Flow B shell
- Flow B input contract
- recurring vs one-off routing
- SharePoint mapping lookup
- existing mapping detection
- existing mapping extraction
- mapped section reuse path
- one-off / no-mapping path
- page / append / persistence control path

Outcome:

Flow B orchestration structure is complete and ready for end-to-end testing.

---

### Build Coach 05 — End-to-End Testing and Controlled Amendment Process

Status: Complete

Scope:

- Stage 8 — End-to-end testing
- UJ1 — one-off single match
- UJ2 — multiple match selection
- UJ3 — recurring meeting with existing mapping
- UJ4 — first-time recurring setup
- UJ5 — no-match recovery
- regression testing
- controlled amendment process

Outcome:

The full solution has a guided end-to-end test and amendment framework.

## Source of truth rule

GitHub is the source of truth.

SharePoint is a Copilot Studio Knowledge mirror.

Copilot Studio implementation must remain aligned to the GitHub baseline.

## Amendment rule

No ad hoc patching is permitted.

Any failure, connector learning, or design correction must be handled through the controlled amendment process.
