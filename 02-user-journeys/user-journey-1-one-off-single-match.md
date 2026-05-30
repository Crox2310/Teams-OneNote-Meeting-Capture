# User Journey 1 — One-Off Meeting, Single Match

## Purpose

UJ1 handles the simplest successful path: Flow A resolves exactly one non-recurring meeting and the topic captures notes to OneNote without recurring setup.

## Entry condition

```text
Topic.FlowAStatus = "SINGLE_MATCH"
Topic.IsRecurring = "false"
Topic.MeetingTitle is not empty
Topic.CalendarEventId is not empty
```

## Behaviour

1. Topic confirms or proceeds with the resolved one-off meeting.
2. Topic applies Outlook Data Capture Profile V1 inclusion flags.
3. Topic generates `PageHtml`.
4. Topic calls Flow B with `IsRecurring = "false"` and `UpdateType = "NOTES"`.
5. Flow B creates or updates the one-off meeting notes page.
6. Topic returns success with `OutPageLink`.

## Guardrails

```text
Do not ask recurring setup questions.
Do not create a recurring mapping.
Do not call Flow B if MeetingTitle or CalendarEventId is blank.
Do not include attachment binary content in V1.
```
