# Project Brief — Teams-OneNote-Meeting-Capture

## Outcome

Create a Copilot Studio agent that helps a user capture meeting notes into OneNote by resolving the correct Outlook / Teams meeting, creating or updating the correct OneNote page, and returning a clean page link.

## Product standard

The OneNote output should be at least as useful as OneNote's Meeting Details function, but cleaner and more action-oriented.

## User-facing principles

- Low friction.
- Clear meeting confirmation.
- No technical metadata on the OneNote page.
- No manual OneNote setup for normal one-off meetings.
- Guided setup only for first-time recurring meetings.
- Created/updated result should be clear.
- A working OneNote link should be returned.

## Current design decision

Option A is the standard:

```text
Guided OneNote section setup applies only to first-time recurring meetings.
One-off meetings use the default meeting notes section unless the user explicitly requests otherwise.
```
