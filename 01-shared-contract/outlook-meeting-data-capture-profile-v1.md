# Outlook Meeting Data Capture Profile V1

## Purpose

Flow A should capture the richest reasonable Outlook event payload available from the connector. The topic decides what to include in the OneNote page using inclusion flags. Full attachment content is not captured in V1.

## Optional V1 outputs

```text
OrganizerSummary
BodyPreview
BodyHtml
OnlineMeetingSummary
HasAttachments
AttachmentSummary
CategoriesSummary
Sensitivity
Importance
ResponseStatus
OutlookRawEventJson
```

## Attachment handling in V1

```text
1. Inspect FA03A_DEBUG_RawConnectorOutput for attachment-related fields.
2. If safe metadata is available, populate HasAttachments and AttachmentSummary.
3. If only an attachment indicator is available, populate HasAttachments only.
4. If no attachment fields are available, leave HasAttachments and AttachmentSummary blank.
5. Do not retrieve or copy attachment binary content in V1.
```
