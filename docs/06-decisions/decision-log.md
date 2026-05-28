# Decision Log

## DL-001 — Guided section setup option

Decision:

```text
Use Option A.
Guided OneNote section setup applies only to first-time recurring meetings.
One-off meetings use a default meeting notes section unless the user explicitly requests another section.
```

Rationale:

- Minimises friction for one-off meetings.
- Asks for setup only when long-term recurring value exists.
- Avoids unnecessary section creation prompts.

## DL-002 — Flow A v1 scope

Decision:

```text
Flow A v1 resolves Outlook meeting selection only.
```

Excluded from Flow A v1:

- OneNote.
- SharePoint.
- Flow B.
- Attendee extraction.
- Recurring setup.

## DL-003 — Attendees deferred to v2

Decision:

```text
AttendeesSummary remains empty string in Flow A v1.
FA10 and FA10A are not built in v1.
```

Rationale:

- Attendee extraction is enrichment.
- Baseline proof is meeting lookup, branching, and string outputs.
- Attendee schema should be confirmed from FA03A debug output first.

## DL-004 — OneNote `/pages` safeguard belongs to Flow B

Decision:

```text
Flow B must normalise section references to TargetSectionPagesUrl ending in /pages before Create page in a section.
```

Rationale:

- Flow A does not touch OneNote.
- The OneNote create-page action needs the section pages endpoint.
