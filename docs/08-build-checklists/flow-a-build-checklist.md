# Flow A Build Checklist

## Before building

- [ ] Confirm flow name: `PA - Resolve Meeting Selection - v1 Clean Build`
- [ ] Confirm this is an Agent Flow in Copilot Studio.
- [ ] Confirm Flow A will use only Office 365 Outlook.
- [ ] Confirm no OneNote or SharePoint actions are added.

## Build discipline

- [ ] Rename every action immediately.
- [ ] Do not create expressions until final action names are set.
- [ ] Do not rename actions after expressions reference them.
- [ ] Use one final Respond to Agent action only.

## Scope exclusions

- [ ] Do not build FA10 / FA10A attendees loop.
- [ ] Do not call Flow B.
- [ ] Do not add recurring setup.
- [ ] Do not create OneNote pages.

## Testing

- [ ] Run standalone test.
- [ ] Inspect FA03A debug output.
- [ ] Confirm FA04 value path.
- [ ] Confirm outputs are strings.
- [ ] Confirm NO_MATCH path.
- [ ] Confirm SINGLE_MATCH path.
- [ ] Confirm MULTIPLE_MATCHES path.
