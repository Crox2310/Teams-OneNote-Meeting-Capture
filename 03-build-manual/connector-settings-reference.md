# Connector Settings Reference

## Flow A — Outlook meeting lookup

Connector: `Office 365 Outlook — Get calendar view of events`

Required debug action: `FA03A_DEBUG_RawConnectorOutput`

## Flow A optional field handling rule

If an optional output field is absent from the Outlook connector output, Flow A must return an empty string `""` for that output.

Do not return `null`. Do not return an object. Use the same coalesce-to-empty-string pattern as core fields.

## Flow B — SharePoint mapping lookup

Action: `FB09_SP_GetItems_BySeriesKey`

Connector: `SharePoint — Get items`

Filter query expression:

```powerautomate
concat('SeriesKey eq ''', triggerBody()?['SeriesMasterId'], '''')
```

## Flow B — OneNote actions

```text
FB15_ON_GetSections
FB18B_NEW_ON_CreateSection
FB20_ON_CreatePage
FB12C_TRUE_ACCESSIBLE_AppendPage
```
