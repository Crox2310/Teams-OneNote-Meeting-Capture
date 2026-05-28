# Architecture Overview

## High-level architecture

```text
User
  ↓
Copilot Studio Topic
  ↓
Agent Flow A — Resolve Meeting Selection
  ↓
Topic confirmation / selection / branching
  ↓
Agent Flow B — Resolve OneNote Section and Capture Notes
  ↓
OneNote page created or updated
  ↓
User receives clean OneNote link
```

## Responsibility split

| Layer | Responsibility |
|---|---|
| Agent | Overall behaviour, instructions, tool availability |
| Topic | User journey, branching, confirmation, multiple-match selection, recurring setup questions |
| Agent Flow A | Outlook meeting lookup only |
| Agent Flow B | OneNote section/page creation, append/update, SharePoint recurring mapping |
| OneNote | Clean human-readable meeting note page |
| SharePoint | Recurring meeting mapping store |

## Core physical Agent Flows

| Agent Flow | Build name | Purpose |
|---|---|---|
| Agent Flow A | `PA - Resolve Meeting Selection - v1 Clean Build` | Outlook meeting lookup only |
| Agent Flow B | `PA - Resolve OneNote Meeting Section and Capture Notes` | OneNote create/update and recurring mapping |

## Optional future Agent Flows

| Optional Agent Flow | Purpose |
|---|---|
| Agent Flow C | Separate recurring setup helper if Flow B becomes too complex |
| Agent Flow D | Post-meeting summary / transcript append |
| Agent Flow E | Diagnostics / connector validation |
