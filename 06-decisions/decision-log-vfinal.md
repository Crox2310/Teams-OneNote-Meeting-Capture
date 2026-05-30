# Decision Log vFinal

```text
1. Flow A owns Outlook meeting lookup and Outlook Data Capture Profile V1.
2. Topic owns conversation, confirmation, branching, validation, inclusion flags, and PageHtml generation.
3. Flow B owns OneNote write/update and SharePoint recurring mapping.
4. UJ3 and UJ4 share one combined Flow B recurring logic design.
5. SeriesMasterId is treated as an opaque key.
6. Blank SeriesMasterId offers one-off fallback.
7. Outlook meeting attachments are not copied as binary content in V1.
8. Attachment summary is allowed in V1 only if safe metadata is visible in Flow A raw output.
9. Full attachment retrieval is deferred to V2 / gated enhancement.
```
