# Project Overview

## Plain English outcome

This project delivers a Copilot Studio agent that helps a user capture meeting notes into OneNote from a Teams/Copilot conversation.

The user asks the agent to capture notes for a meeting. The agent searches the user's Outlook calendar, resolves the correct meeting, confirms or disambiguates where required, then creates or updates a OneNote page with a consistent meeting-note template.

For recurring meetings, the agent remembers where future notes for the same recurring series should go by using a SharePoint mapping list. If the recurring meeting has already been set up, notes are appended to the existing OneNote page. If it is the first time the recurring meeting has been used, the agent guides the user through choosing an existing OneNote section or creating/reusing a new section.

## Outlook Data Capture Profile V1

Flow A should retrieve and preserve the richest reasonable Outlook event payload available from the connector debug output, but the topic decides what is included in the OneNote page.

```text
- Core meeting data is captured by default.
- Optional Outlook data is captured where available but can be turned off in PageHtml.
- Full attachment content is not captured in V1.
- Attachment summary is included only if Outlook connector output exposes safe metadata or an attachment indicator.
- Full attachment retrieval is deferred to V2 / gated enhancement.
```
