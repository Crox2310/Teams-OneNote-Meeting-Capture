# End-to-End Test Matrix

## Outlook Data Capture Profile V1 tests

| Test | Meeting data | Expected result |
|---|---|---|
| OD-V1-01 | Simple meeting | Core fields populated; optional outputs blank |
| OD-V1-02 | Meeting with body | Body output populated if available; not included unless flag true |
| OD-V1-03 | Meeting with attendees | Compact `AttendeesSummary` populated |
| OD-V1-04 | Recurring meeting | `IsRecurring` and `SeriesMasterId` populated where available |
| OD-V1-05 | Teams meeting details | `OnlineMeetingSummary` populated if available; off by default |
| OD-V1-06 | Meeting with small attachment | `HasAttachments` or `AttachmentSummary` populated only if visible in raw output; no binary capture |
| OD-V1-07 | Meeting with document link in body | Link remains body content; not treated as Outlook attachment unless connector exposes attachment metadata |
| OD-V1-08 | Inclusion flag evaluation: `Topic.IncludeOrganizer = "false"` while `OrganizerSummary` is populated | `OrganizerSummary` is absent from `PageHtml`. |

## UJ1 named test cases

| Test | Scenario | Expected result |
|---|---|---|
| UJ1-T01 | Single non-recurring meeting, all fields populated | Topic generates PageHtml, calls Flow B once, and returns OneNote page link. |
| UJ1-T02 | Single non-recurring meeting, Start and End empty | PageHtml degrades safely and uses configured fallback time text. |
| UJ1-T03 | Single non-recurring meeting, Location empty | Location paragraph is omitted from PageHtml. |
