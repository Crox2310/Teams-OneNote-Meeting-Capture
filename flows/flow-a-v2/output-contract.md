# Flow A v2 — Output Contract

**Locked:** 26 September 2026

## Response fields

| Field key | Name | Type | No-match value | Single-match value | Multi-match value |
|---|---|---|---|---|---|
| `status` | OutStatus | string | `NO_MATCH` | `MULTIPLE_MATCHES` | `MULTIPLE_MATCHES` |
| `matchcount` | OutMatchCount | string | `'0'` | `'1'` | count as string |
| `candidatelist` | OutCandidateList | string | `''` | `''` | formatted list with capture status |
| `meetingtitle` | OutMeetingTitle | string | `''` | meeting subject | `''` |
| `calendareventid` | OutCalendarEventId | string | `''` | event id | `''` |
| `isrecurring` | OutIsRecurring | string | `''` | `'true'` or `'false'` | `''` |
| `seriesmasterid` | OutSeriesMasterId | string | `''` | seriesMasterId or `''` | `''` |
| `onlinemeetingurl` | OutOnlineMeetingUrl | string | `''` | joinUrl or `''` | `''` |
| `bodypreview` | OutBodyPreview | string | `''` | body preview (stripped) or `''` | `''` |
| `endtime` | OutEndTime | string | `''` | end time (UTC flat string) or `''` | `''` |

## Notes
- All fields use `coalesce()` in the Response action to select the correct branch value with an empty string fallback.
- `endtime` is added to v2 contract (was `text_6` passthrough in v1 via C6D — confirming it appears in the output contract here for completeness).
- Single-match returns `candidatelist=''` and the meeting details directly. The Topic's C-node handling for single-match (direct confirm without list display) depends on this — do not change.
- Multi-match returns meeting details as empty strings and the list in `candidatelist`. The Topic displays the list and waits for selection.
