# Decision Log Addendum — UJ2 Option A

## DL-005 — User Journey 2 selection resolution

Decision:

```text
Adopt Option A for User Journey 2.
```

Option A means:

```text
The topic calls Flow A once to display multiple candidate meetings.
The user selects a number.
The topic validates the selected number.
The topic calls Flow A a second time with InSelectedNumber.
Flow A resolves the selected number to one selected meeting and returns SINGLE_MATCH outputs.
```

## Rationale

Option A keeps selected-meeting resolution inside Flow A, where the Outlook connector output and candidate array logic already exist.

Option B, hidden `CandidatePayloadJson`, was rejected for v1 because it would require brittle JSON handling in the Copilot Studio topic.

Option C, fixed Candidate1/Candidate2 fields, was rejected for v1 because it creates a large and brittle output contract.

## Contract change

Add this Flow A input for v3.2:

| Input | Type | Value type | Source | Notes |
|---|---|---|---|---|
| `InSelectedNumber` | Text | Dynamic content or Plain text | User selected number | Blank on first call. Populated on second call for UJ2 selected-number resolution. |

## Topic validation requirement

Before the second Flow A call, the topic must validate:

```text
Topic.SelectedNumber is not blank
Topic.SelectedNumber is numeric
Topic.SelectedNumber >= 1
Topic.SelectedNumber <= Topic.MatchCount
```

If validation fails, the topic must not call Flow A again and must not call Flow B.

## Risk mitigation

Risk:

```text
The candidate array is rebuilt on the second Flow A call.
A calendar change between calls could shift candidate order.
```

Mitigation:

```text
After the second Flow A call, the topic must show the resolved meeting title and time to the user for confirmation before calling Flow B.
```
