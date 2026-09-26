# Flow A v2 — Trigger Contract

**Flow name:** PA - Resolve Meeting Selection - v2
**Trigger type:** Request (Skills kind — When an agent calls the flow)
**Locked:** 26 September 2026

## Input fields

| Key | Name | Type | Required | Notes |
|---|---|---|---|---|
| `text` | InSelectedNumber | string | Yes | Always `'NONE'` in current Topic — meeting selection is handled Topic-side via ParseJSON(CandidatesJson). The IsSelectionMode branch that read this was confirmed dead code in the existing flow and is NOT rebuilt in v2. Field kept in contract because the Topic still sends it. |
| `text_1` | DateContext | string | Yes | ISO date string (yyyy-MM-dd) for the day to browse meetings. If empty, flow defaults to today via utcNow(). |

## Key assignment note

The new Copilot Studio agent flow Designer assigns input keys sequentially (`text`, `text_1`, `text_2`...) in the order inputs are added. v2 uses `text` and `text_1` because only two inputs are needed and no gaps are required.

This differs from v1 which used `text_1` and `text_3` (gaps existed because inputs were added at different build stages). At Topic switchover, the C2 call must be updated to pass `text` for InSelectedNumber and `text_1` for DateContext.

## Expression reference

All expressions in the flow that read trigger inputs use:

| Value needed | Expression |
|---|---|
| InSelectedNumber | `triggerBody()?['text']` |
| DateContext | `triggerBody()?['text_1']` |
