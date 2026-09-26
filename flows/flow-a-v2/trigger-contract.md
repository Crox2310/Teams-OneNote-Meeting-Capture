# Flow A v2 — Trigger Contract

**Flow name:** PA - Resolve Meeting Selection - v2
**Trigger type:** Request (Skills kind — When an agent calls the flow)
**Locked:** 26 September 2026

## Input fields

| Key | Name | Type | Required | Notes |
|---|---|---|---|---|
| `text_1` | InSelectedNumber | string | Yes | Always `'NONE'` in current Topic — meeting selection is handled Topic-side via ParseJSON(CandidatesJson). The IsSelectionMode branch that read this was confirmed dead code in the existing flow and is NOT rebuilt in v2. Field kept in contract because the Topic still sends it. |
| `text_3` | DateContext | string | Yes | ISO date string (yyyy-MM-dd) for the day to browse meetings. |

## Notes
- `text_1` = `'NONE'` always. The v1 flow's IsSelectionMode branch (FA15–FA26) that handled non-NONE values is confirmed dead and is not rebuilt. If the Topic contract ever changes to pass a real selection number, this contract must be revisited.
- `text_3` drives all date calculations. If empty, the flow defaults to today via `utcNow()`.
