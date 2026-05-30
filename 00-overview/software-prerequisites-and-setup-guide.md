# Software Prerequisites and Setup Guide

## Required components

| Component | Required for | Expected setup position |
|---|---|---|
| Copilot Studio | Agent and topic | Access required before build |
| Power Automate | Flow A and Flow B | Access required before build |
| Outlook / Exchange calendar | Meeting lookup | User calendar must exist |
| OneNote for Business | Meeting notes pages | Notebook must exist or be created |
| SharePoint / Microsoft Lists | Recurring mapping store | Site access required; list is created during setup |
| Microsoft 365 connector connections | Outlook, OneNote, SharePoint access | Must be authorised during build |

## SharePoint list to create

List name: `RecurringMeetingSectionMap`

| Column display name | Type |
|---|---|
| SeriesKey | Single line of text |
| MeetingTitle | Single line of text |
| NotebookId | Single line of text |
| SectionName | Single line of text |
| SectionPagesUrl | Single line of text |
| PageSelfUrl | Single line of text |
| PageLink | Single line of text |
| Status | Single line of text |
| LastUpdated | Date and Time |

Use single-word column names so internal names remain predictable.
