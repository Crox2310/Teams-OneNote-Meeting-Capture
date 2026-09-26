# Flow B v2 — Split Architecture (Parent + Two Children)

**Design locked:** 26 September 2026
**Author:** David Croxson (design pass via Opus)
**Status:** Contracts + Scope maps locked. **No build steps yet.** Build is Sonnet work.

---

## Why a split

Flow B v1 (`PA - Meeting Capture - B - Resolve OneNote Section`) carries the whole capture pipeline — recurring/one-off routing, SharePoint mapping, OneNote section resolution, page create/append, SharePoint write-back, 6-value status — in one flow built on `SetVariable`/`InitializeVariable`. That is exactly the action class the platform corruption pattern wipes (15-33 actions at once, 12+ incidents).

v2 splits into **one lightweight parent** and **two purpose-built children**:

- The **parent** keeps the v1 trigger byte-identical, routes on `empty(SeriesMasterId)`, calls one child, relays the child output back unchanged. No SharePoint, no OneNote, no variables. Skills trigger + Skills "Respond to the agent" (unchanged from v1).
- The **recurring child** keeps the full section-resolution + mapping logic.
- The **one-off child** is much lighter: the fixed One-Off Meetings section removes all section resolution.

Both children are PowerAppV2-triggered, end in "Respond to a PowerApp or Flow", and return an **identical 16-field output**, so the parent is a pure pass-through.

### Central refactor — zero SetVariable, zero accumulation loops

Every v1 pattern of *Apply-to-each over a single-row filter result → SetVariable* collapses to `first(body('Filter_...'))?['field']` in a Compose. Ambiguity (>1 match) is detected with `length()`, not by looping. Target state per flow: **zero SetVariable, zero accumulation loops, ideally zero InitializeVariable.** If any InitializeVariable proves unavoidable it goes top-level, before the outer Scope (platform constraint: InitializeVariable cannot nest).

### Trigger inputs re-exposed as Composes (corruption + remap safety)

Each child renumbers its trigger keys, so v1 `text_5` (OccurrenceDate) becomes a child `text_5` meaning something else (JoinUrl). To kill that silent-remap trap, every child's Normalize scope re-exposes each trigger input as a named Compose (`NZ00a..`) at the very top, and **all** downstream expressions reference the Compose, never a raw `triggerBody()?['text_N']`. Port v1 logic by action-name, not by key number.

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

1. **Route on `empty(SeriesMasterId)`** (`empty(triggerBody()?['text_2'])`). `IsRecurring` (`text`) is vestigial *for routing* and dropped from both child trigger contracts. Still echoed faithfully in the parent output (`outisrecurring`).
2. **Uniform 16-field child output contract**; parent is a pure pass-through relay.
3. **Graceful child-failure handling** — parent relay coalesces a failed/absent child body to `outstatus = 'ERROR'`.
4. **Write-back on any page CREATE** (new occurrence *or* recovery-created page on an existing mapping), targeting the resolved row ID. Not on a pure append. (Refined during the 26 Sep recheck — see finding below.)
5. **`STALE_MAPPING` kept in the one-off status enum** for uniformity, though unreachable (fixed section never blank).
6. **Recurring section prefix `Mtg -`** — confirmed live 26 Sep.
7. **`outbranchresult` replicates the v1 binding** (mirrors match count) for zero switchover behaviour change.

---

## Findings from the 26 Sep recheck (all folded into the locked files)

- **STALE_MAPPING was a v1 bug**, not a feature: the existing-section guard never set a page action, so successful existing-section captures were mislabelled `STALE_MAPPING`. v2 records `outpageaction` on every path and redefines `STALE_MAPPING` as a genuine stale row (matched mapping, blank `SectionPagesUrl` — the UJ3b condition). Confirm the Topic's user-facing message for STALE vs SUCCESS before switchover.
- **Trigger-key renumbering trap** — neutralised by the Normalize input-Compose rule above.
- **Response kind** — parent = Skills "Respond to the agent"; children = "Respond to a PowerApp or Flow." Do not mix.
- **Relay null-safety** — the 16 relay fields carry a `''` terminal fallback; `outstatus` carries `'ERROR'`.
- **Two open confirmations for the build session:** (a) the recovery-create write-back fix (Decision 4 refined — v1 doesn't do it); (b) `Compose_AgentResponseSummary`'s expression must be pulled from live v1 (not in the known-good reference; may be user-facing).

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
2. **Sonnet, per flow** — one-off and recurring are independent. Suggested: parent → one-off (simplest, proves the child-flow plumbing) → recurring.
3. **Before the recurring build:** pull the full v1 recurring-branch Peek Code (structure/runAfter, not just values) and `Compose_AgentResponseSummary`.
4. Push each Scope's Peek Code to `scope-peek-codes/` immediately after it confirms green.

**Child-flow plumbing prerequisite (both children):** PowerAppV2 trigger + terminal "Respond to a PowerApp or Flow" + "Use this connection" set for every connector in Run-only-users + child must be solution-aware to appear in the Run-a-Child-Flow picker.
