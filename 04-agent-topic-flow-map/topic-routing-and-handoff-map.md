# Topic Routing and Handoff Map

```text
FlowAStatus = SINGLE_MATCH
    → Apply Outlook Data Capture Profile V1 outputs and inclusion flags
    → If IsRecurring = "false" → UJ1
    → If IsRecurring = "true" and SeriesMasterId populated → UJ3
    → If IsRecurring = "true" and SeriesMasterId blank → UJ4 blank-key fallback

FlowAStatus = MULTIPLE_MATCHES → UJ2
FlowAStatus = NO_MATCH → UJ5
FlowAStatus = ERROR → Safe error, no Flow B
```
