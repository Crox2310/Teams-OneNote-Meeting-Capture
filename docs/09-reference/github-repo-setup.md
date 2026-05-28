# GitHub Repository Setup Guide

## Recommended repository name

```text
Teams-OneNote-Meeting-Capture
```

## Recommended structure

```text
Teams-OneNote-Meeting-Capture/
├── README.md
├── AGENTS.md
├── CLAUDE.md
├── .github/
│   ├── copilot-instructions.md
│   └── instructions/
├── docs/
│   ├── 00-project/
│   ├── 01-architecture/
│   ├── 02-user-journeys/
│   ├── 03-agent-flows/
│   ├── 04-topic-design/
│   ├── 05-testing/
│   ├── 06-decisions/
│   ├── 07-claude-review-prompts/
│   ├── 08-build-checklists/
│   └── 09-reference/
```

## Where to place files

- Put `README.md` in the repository root.
- Put `.github/copilot-instructions.md` in the `.github` folder at the repository root.
- Put `AGENTS.md` and `CLAUDE.md` in the repository root.
- Put all design artefacts under `docs/`.

## Why this structure helps Copilot

GitHub supports repository-wide custom instructions in `.github/copilot-instructions.md`.
GitHub also supports path-specific custom instructions under `.github/instructions` and agent instructions using `AGENTS.md` files.

This repository keeps the main build artefacts in Markdown so that people and AI assistants can read them consistently.
