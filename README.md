# Teams-OneNote-Meeting-Capture

## Plain English outcome

Teams-OneNote-Meeting-Capture is a Copilot Studio and Power Automate design for capturing meeting notes into OneNote.

The agent searches Outlook calendar data, resolves the correct meeting, handles one match, multiple matches, no match, and recurring meeting scenarios, then creates or updates a OneNote page. For recurring meetings, a SharePoint mapping list stores the relationship between the recurring series and the OneNote location so future notes go to the correct place.

## Claude final review confidence

```text
Design confidence: 91%
First-build confidence: 83%
Production confidence after validation: 80%
```

Claude recommended proceeding to build after four amendments. Those amendments are applied in this complete final baseline.

## Important build note

The design is complete, but Flow B recurring build must not start until connector validation gates are passed.
