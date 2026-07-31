# Topic YAML Review — 31 July 2026

**Purpose:** review notes on `topic-export-2026-07-31.yaml`, captured as part of full end-to-end material gathering (Flow A, Flow B, Topic YAML) to support a proper design for the outstanding `OutStatus` differentiation work and the one-off existing-page resolution gap found on 30 July.

**Status:** Topic YAML received and reviewed. Flow A and Flow B full exports not yet received — this review is Topic-only and should be revisited once those arrive.

## Confirms existing understanding

- `OutStatus` handling is exactly as the 20 July gap-analysis described: a single `ConditionGroup` checking `Topic.OutStatus = "OK"`, with everything else falling into one generic error message ("I'm sorry, something went wrong..."). No `SectionRetryCount`, no per-value routing.
- P/N/date-jump navigation (`C6_Check_Input`, `C6B_Check_N`, `C6C_Check_Date`) is present and matches the 20 July date-jump feature doc. No reword/retry option, no explicit Stop action — matches the still-open UJ5 gap.
- Flow A is called twice: once blank (`C2_Call_FlowA_Initial`) and once with the user's selection (`invokeFlowAction_bIIKPf`) — matches UJ2's documented mechanism exactly.

## New findings, relevant to the one-off existing-page gap (30 July)

**1. `CalendarEventId` is already passed into Flow B.**

`C10_Call_FlowB_Create_Page`'s input binding includes:
```yaml
text_4: =Topic.CalendarEventId
```
alongside `IsRecurring`, `MeetingTitle`, and `SeriesMasterId`. This value is sourced from Flow A's output (`calendareventid: Topic.CalendarEventId`, set on both Flow A calls) and is available for **every** meeting, recurring or one-off — unlike `SeriesMasterId`, which is blank/absent for one-off meetings.

**Why this matters:** the 30 July doc's open design question asked how a one-off meeting's existing page could be resolved on recapture, given there's no `SeriesMasterId` to key on. `CalendarEventId` is a candidate answer to that question, and — notably — the plumbing to get it into Flow B **already exists**. This doesn't confirm Flow B does anything with it (yesterday's trace never encountered `text_4`/`CalendarEventId` referenced anywhere inside Flow B's actions), but it means if a one-off mapping mechanism is the chosen design, at least one piece of it (getting a stable unique key to Flow B) is already built, rather than needing new Topic-side work.

**Needs confirming against the Flow B export:** does Flow B currently read `text_4` / `CalendarEventId` for anything at all? If not, it's an unused trigger input sitting there ready to be wired up.

**2. Flow B already has an output field for exactly the value we found broken.**

`C10_Call_FlowB_Create_Page`'s output binding includes:
```yaml
outexistingpageselfurl: Topic.OutExistingPageSelfUrl
```
This confirms Flow B's "Respond to the agent" action has a field explicitly mapped to send an existing-page self-URL back to the Topic — almost certainly sourced from `varFinalExistingPageSelfUrl`, the variable confirmed `null` throughout the 30 July failing run. The Topic receives this value but currently does nothing with it (not referenced in any condition or message) — it's captured into `Topic.OutExistingPageSelfUrl` and then unused.

**Needs confirming against the Flow B export:** the exact "Respond to the agent" mapping, to see definitively whether `outexistingpageselfurl` sources from `varFinalExistingPageSelfUrl` as assumed, and whether any other one-off-relevant outputs (`outpagedecision`, `outmatchcount`, `outbranchresult`) shed further light on the intended one-off design.

## Other observations (lower priority, noted for completeness)

- `Topic.PageTitle` construction differs by `IsRecurring`: recurring pages are titled by date only; one-off pages are titled `"<date> - <meeting title>"`. Not related to the current bug, but relevant context if the one-off fix ends up needing to match pages by title within a section (one of the two candidate designs from 30 July) — the title format is already meeting-specific for one-off pages, which would help a title-based match be more reliable.
- The full HTML page body sent to Flow B (`text_3` on `C10_Call_FlowB_Create_Page`) is constructed entirely in the Topic, not Flow B — worth remembering when reasoning about `Compose_UpdateHtmlFragment` and related Flow B actions, since the content itself originates upstream.

## Next steps

1. Awaiting Flow A full export (JSON) and Flow B full export (JSON) from David.
2. Once received: confirm whether Flow B reads `text_4`/`CalendarEventId` anywhere, and pull the exact "Respond to the agent" mapping for `OutExistingPageSelfUrl` and related one-off outputs.
3. Request SharePoint mapping list column schema (full column list, not just the three columns seen so far) to evaluate whether the existing mapping table could be reused with `CalendarEventId` as an additional/alternative key for one-off meetings, rather than requiring a wholly new mechanism.
4. Once all material is in: produce a single end-to-end design doc proposing a specific fix for the one-off gap, informed by whichever of the two candidate approaches (extend section-resolution loop vs. new one-off mapping) the fuller picture supports — flagging clearly which parts are confirmed vs inferred.
