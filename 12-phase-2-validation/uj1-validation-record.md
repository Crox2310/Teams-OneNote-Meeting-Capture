# UJ1 Validation Record — One-off Single Match

## User journey

UJ1 — one-off single match

## Purpose

Validate that the user-facing Meeting Capture solution can resolve a single one-off Outlook meeting and create a OneNote meeting notes page without creating a recurring SharePoint mapping.

## Expected behaviour

- Flow A returns SINGLE_MATCH.
- MatchCount equals 1.
- MeetingTitle is populated.
- CalendarEventId is populated.
- IsRecurring is false.
- Topic routes to SINGLE_MATCH.
- Flow B follows the one-off path.
- OneNote page is created in the default meeting notes section.
- No recurring SharePoint mapping is created.
- Agent returns a clear success response.

## Test input

Record the exact user prompt used here:

```text
[Not yet tested — inspection stopped before live UJ1 execution]
```

## Inspection status

Status:

```text
NOT READY / FAILED INSPECTION
```

## Inspection finding

During Phase 2 inspection, the current user-facing Meeting Capture topic was reviewed before running a live UJ1 test.

The topic inspected was:

```text
Meeting Capture (v3 rebuild)
```

within the agent:

```text
Teams → OneNote Meeting Capture
```

The topic currently uses the trigger:

```text
A message is received
```

The expected user-facing trigger phrases were not present, including:

```text
Capture meeting notes
Create meeting notes
Take notes for my meeting
Meeting capture
```

The initial Flow A call is present:

```text
A1_Call_ToolA_InitialLookup
```

However, the visible Flow A inputs are currently hardcoded test values rather than dynamic user input.

Observed Flow A input values:

```text
SearchText = test
StartDateTime = 2026-05-17T00:00:00
EndDateTime = 2026-05-17T00:00:00
SelectedNumber = ""
```

## Impact on UJ1 validation

UJ1 cannot yet validate real user request capture or dynamic meeting search behaviour.

Because the topic is using hardcoded test values, the current topic cannot prove the intended UJ1 path:

```text
User prompt
→ dynamic meeting search text
→ Flow A SINGLE_MATCH
→ SINGLE_MATCH topic routing
→ Flow B one-off path
→ OneNote page creation
→ no recurring SharePoint mapping
```

## Failure classification

Failure layer:

```text
Topic
```

Failure type:

```text
Topic trigger / input capture / dynamic content mapping
```

## Current assessment

This is not yet a Flow A failure.

This is not yet a Flow B failure.

The current finding is a topic readiness issue.

The current topic appears to be a rebuild or test harness rather than a fully user-facing UJ1-ready Meeting Capture topic.

Before live UJ1 validation can proceed, we need to confirm whether:

1. Meeting Capture (v3 rebuild)` is the intended production Meeting Capture topic, or
2. another production-ready Meeting Capture topic exists, or
3. this topic must be amended into a user-facing UJ1-ready topic.

## Test meeting

Record once live UJ1 testing begins:

- Meeting title:
- Meeting date/time:
- Meeting type: one-off
- Calendar used:
- Expected OneNote section:

## Evidence captured

- Screenshot of current topic state: Yes
- Flow A call visible: Yes
- Hardcoded Flow A inputs visible: Yes
- Flow A run reference: Not yet tested
- Flow B run reference: Not yet tested
- OneNote page link: Not yet created
- SharePoint mapping checked: Not yet tested
- Transcript captured: Not yet tested

## Result

```text
NOT READY / FAILED INSPECTION
```

## Actual behaviour

Topic inspection found:

- The topic exists.
- Flow A is called through `A1_Call_ToolA_InitialLookup`.
- The visible Flow A call uses hardcoded test values.
- User-facing trigger phrases are not present.
- Dynamic user search text capture is not yet confirmed.
- UJ1 live validation should not proceed until topic readiness is clarified.

## Notes

This inspection confirms that the current issue is with topic readiness, not with Flow A or Flow B execution.

No implementation changes should be made informally.

The next step is to determine whether this topic is the intended production path or whether a separate production-ready topic exists.

If `Meeting Capture (v3 rebuild)` is confirmed as the intended production topic, a controlled amendment should be raised before changing the topic trigger, user input capture, or Flow A input mappings.

## Controlled amendment required

Current status:

```text
To be assessed
```

Reason:

A controlled amendment may be required if this topic is confirmed as the intended production Meeting Capture topic and must be updated to support dynamic user-facing UJ1 execution.

Potential amendment area:

- Meeting Capture topic trigger
- User input capture
- Flow A input mapping
- UJ1 test readiness
- Documentation / validation record

## Next action

Check whether another production-ready Meeting Capture topic exists in the `Teams → OneNote Meeting Capture` agent.

If no other production-ready topic exists, raise a controlled UJ1 topic-readiness amendment before changing the topic.
