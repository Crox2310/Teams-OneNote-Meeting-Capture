# Agent Flow B — OneNote Capture Design Notes

## Purpose

Flow B creates or updates the OneNote meeting page and handles recurring meeting storage.

## Scope

Flow B owns:

- OneNote section/page resolution.
- Create page in section.
- Update/append existing page.
- SharePoint recurring meeting mapping.
- First-time recurring setup execution.
- OneNote `/pages` normalisation.

## OneNote `/pages` safeguard

Before any OneNote Create page in a section action, Flow B must normalise the target section reference into `TargetSectionPagesUrl`.

`TargetSectionPagesUrl` must end with `/pages`.

## Safe normalisation expression

Value type: **Expression**

```powerautomate
if(
  endsWith(outputs('Compose_SectionSelfUrl'), '/pages'),
  outputs('Compose_SectionSelfUrl'),
  concat(outputs('Compose_SectionSelfUrl'), '/pages')
)
```

## Create vs update distinction

```text
Create new page:
Use TargetSectionPagesUrl ending in /pages.

Update existing page:
Use PageSelfUrl / PageId required by the update action.
```

## Recurring setup decision

Guided setup applies only to first-time recurring meetings.
One-off meetings do not prompt for section setup by default.
