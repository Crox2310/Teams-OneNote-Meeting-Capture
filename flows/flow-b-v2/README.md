# Flow B v2 — Split Architecture (Parent + Two Children)

**Design locked:** 26 September 2026
**Author:** David Croxson (design pass via Opus)
**Status:** Contracts + Scope maps locked. **No build steps yet.** Build is Sonnet work.

---

## Why a split

Flow B v1 (`PA - Meeting Capture - B - Resolve OneNote Section`) carries the whole capture pipeline — recurring/one-off routing, SharePoint mapping, OneNote section resolution, page create/append, SharePoint write-back, 6-value status — in one flow built on `SetVariable`/`InitializeVariable`. That is exactly the action class the platform corruption pattern wipes (15-33 actions at once, 12+ incidents).

v2 splits into **one lightweight parent** and **two purpose-built children**:

- The **parent** keeps the v1 trigger byte-identical, routes on `empty(SeriesMasterId)`, calls one child, relays the child output back unchanged. No SharePoint, no OneNote, no variables.
- The **recurring child** keeps the full section-resolution + mapping logic.
- The **one-off child** is much lighter: the fixed One-Off Meetings section removes all section resolution.

Both children return an **identical 16-field output**, so the parent is a pure pass-through.

### Central refactor — zero SetVariable, zero accumulation loops

Every v1 pattern of *Apply-to-each over a single-row filter result → SetVariable* collapses to `first(body('Filter_...'))?['field']` in a Compose. Ambiguity (>1 match) is detected with `length()`, not by looping. Target state per flow: **zero SetVariable, zero accumulation loops, ideally zero InitializeVariable.** If any InitializeVariable proves unavoidable it goes top-level, before the outer Scope (platform constraint: InitializeVariable cannot nest in a Scope or Condition).

---

## The three flows

| Flow | Working name | Role | Weight |
|---|---|---|---|
| Parent | `PA - Meeting Capture - B - Router - v2` | Trigger contract, route, relay | ~1/4 session |
| Recurring child | `PA - Meeting Capture - B - Recurring - v2` | Mapping + section + page + write-back | ~1.5 sessions |
| One-off child | `PA - Meeting Capture - B - OneOff - v2` | Mapping + page + write-back (fixed section) | ~1/2-1 session |

The one-off child shares nothing with the recurring child except the output contract, so it can be built and tested end-to-end independently.

---

## Decisions taken (defaults, 26 Sep 2026)

1. **Route on `empty(SeriesMasterId)`** (`empty(triggerBody()?['text_2'])`). `IsRecurring` (`text`) is vestigial *for routing* and dropped from both child trigger contracts. It is still echoed faithfully in the parent output (`outisrecurring`).
2. **Uniform 16-field child output contract**; parent is a pure pass-through relay.
3. **Graceful child-failure handling** — parent relay coalesces a failed/absent child body to `outstatus = 'ERROR'` so the Topic always receives a clean status.
4. **Write-back only on new page / new mapping row.** A pure append to an already-mapped page does not re-MERGE `JoinUrl`/`EndTime`. (Confirm against v1 during build.)
5. **`STALE_MAPPING` kept in the one-off status enum** for uniformity, though it is judged unreachable there (fixed section always recovers via create).
6. **Recurring section prefix `Mtg -`** (per 21 Sep known-good). `Rec -` rename not confirmed shipped; lock to live value at build time.
7. **`outbranchresult` replicates the v1 binding** (bound to match count) for zero switchover behaviour change. Rebinding to a real branch label is a separate follow-up; keys stay locked either way, so a value fix cannot break C10.

---

## Key constants (carried in)

**SharePoint mapping list** (`RecurringMeetingSectionMap`)
- dataset: `https://jsainsbury.sharepoint.com/sites/coplt`
- table GUID: `186b3c9f-e758-4e85-83d5-685946614a0a`
- `Get items` $top: `500`

**Notebook key — standard (all Flow B v2 OneNote actions)**
```
Meeting Notes|$|https://jsainsbury-my.sharepoint.com/personal/david_croxson_sainsburys_co_uk/Documents/Meeting Notes
```

**Fixed One-Off Meetings section (one-off child only)**
```
https://www.onenote.com/api/v1.0/myOrganization/siteCollections/b5f8860c-4772-4e8b-b340-e80ba9d490fa/sites/d814850f-59bb-4182-92b7-e25d8c6a0487/notes/sections/1-cbb5e863-2bd7-458f-9944-4fec9d20607b/pages
```

---

## Peek Code / health-check strategy (methodology principle 4)

Each flow is wrapped in **one outer Scope** containing inner Scopes by phase:
- Outer Scope Peek Code = full-flow health check in a single pull.
- Any one inner Scope Peek Code = that phase in a single pull.

Outer Scope names: `Scope_FlowB_Parent`, `Scope_FlowB_Recurring`, `Scope_FlowB_OneOff`. Session-start protocol per flow: open, wait 20s untouched, Flow Checker, then Peek Code the outer Scope and cross-reference known-good before touching anything.

---

## Build sequencing

1. **Now (Opus):** contracts + Scope maps locked (this folder). ✔
2. **Sonnet, per flow, in any order** — one-off child and recurring child are independent. Suggested: parent → one-off (simplest, proves the child-flow plumbing) → recurring.
3. Push each Scope's Peek Code to `scope-peek-codes/` immediately after it confirms green.

**Child-flow plumbing prerequisite (both children):** PowerAppV2 trigger + terminal Respond action + "Use this connection" set for every connector in Run-only-users + child must be solution-aware to appear in the Run-a-Child-Flow picker.
