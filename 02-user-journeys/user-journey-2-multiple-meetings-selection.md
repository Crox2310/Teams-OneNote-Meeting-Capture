# User Journey 2 — Multiple Meetings, User Selects One

## Purpose

UJ2 handles the case where Flow A finds multiple candidate meetings and the user must select one.

## Accepted design

```text
Option A — second Flow A call with InSelectedNumber
```

## Key rules

- First Flow A call uses `InSelectedNumber = ""`.
- Topic displays `CandidateList`.
- Topic captures `Topic.SelectedNumber` using Number entity.
- Topic validates range and integer guard.
- Topic passes `InSelectedNumber = Text(Topic.SelectedNumber)` on the second Flow A call.
- Flow A returns `SINGLE_MATCH`, `MatchCount = "1"`, `CandidateList = ""`, and Outlook Data Capture Profile V1 outputs for the selected meeting where available.

## Downstream routing

```text
IsRecurring = "false" → UJ1
IsRecurring = "true" and SeriesMasterId populated → UJ3
IsRecurring = "true" and SeriesMasterId blank → UJ4 blank-key fallback
```
