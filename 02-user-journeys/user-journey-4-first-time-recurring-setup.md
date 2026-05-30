# User Journey 4 — First-Time Recurring Meeting Setup

## Purpose

UJ4 handles first-time recurring meeting setup. It asks where future notes should be stored, resolves or creates the OneNote section, creates the meeting page, and persists the recurring mapping.

## Outlook data profile

UJ4 uses Outlook Data Capture Profile V1 when generating PageHtml. Core meeting data is included; optional sections are controlled by inclusion flags. Attachment content is not captured in V1.

## Blank SeriesMasterId fallback

Do not create a recurring mapping with blank `SeriesMasterId`. Offer:

```text
1. Capture as a one-off note now
2. Skip for now
```

## Setup choice

```text
1. Use an existing OneNote section
2. Create a new OneNote section
```

## Retry control

`Topic.SectionRetryCount` must be initialised to `0`. It may reach a maximum value of `1` before graceful exit.

## Flow B outcomes

```text
SUCCESS → full success branch
PARTIAL_SUCCESS → notes captured but mapping failed
SETUP_SECTION_NOT_FOUND → re-prompt once
SETUP_SECTION_AMBIGUOUS → re-prompt once
ERROR → safe error
```
