# Performance Analysis — Phase 3 Candidate: Architecture & Latency Review

**Status:** Not started. This is a scoping doc only — no changes have been made to Flow A, Flow B, or the Topic as a result of this analysis. Created 23 August 2026 at David's request, following a "just curious" conversation about whether the architecture could be laid out differently for speed.

**Purpose:** define what to look at and how to collect the evidence, ahead of David running the agent through a normal working week to gather real usage and timing data. Analysis and any resulting design work happens in a future session, once that data exists.

---

## Why this is scoped as data-collection-first, not a redesign

The initial conversation surfaced plausible candidates for where latency might live, but **all of it was inference from whole-run durations** (the `00:00:17`-style totals visible in Flow overview run history), not from a breakdown of which individual action consumes how much time. Proposing an architecture change from that would repeat exactly the mistake the working method spent all of 23 August avoiding — reasoning from a guess instead of evidence. This doc exists to make sure that discipline carries into performance work too.

---

## What we already know (fact, not inference)

- **`Delay_Post_Page_Creation`** in Flow B is a **fixed, deliberate 5-second wait**, inserted because newly created OneNote pages aren't reliably queryable via `Get_Pages_In_Section` immediately after `CreatePageInSection` returns. This is a genuine, currently-necessary cost — not incidental waste. Any performance work needs to treat this as a floor, not a target, unless a different confirmation mechanism is found that doesn't rely on a fixed wait.
- **The architecture is fully sequential across three layers** for every single turn: Topic → Flow A (search/disambiguate) → Topic (confirm) → Flow B (create/update page) → Topic (report). There is no parallelism between Flow A and Flow B, and no step is currently async from the user's perspective.
- **`FA08_Get_calendar_view_of_events` (the Graph calendar call) re-runs on every P/N/date navigation keypress**, not just on the initial search. Each "N" triggers the full Topic → Flow A round trip again — new date range, new Graph call, `FA09B` filter, `FA09C` sort — with no caching of a previously-fetched day within the same conversation.
- **Flow A's filter/sort chain (`FA09` → `FA09B` → `FA09C`) always runs in full**, even when nothing about the calendar could plausibly have changed since an identical call earlier in the same session.

## What we do NOT yet know (needs real data)

- Which individual actions inside Flow A and Flow B account for the largest share of a run's total duration — Graph API calls, OneNote API calls, SharePoint calls, or Topic-level round-trip overhead between the three `InvokeFlowAction` hops.
- Whether latency is dominated by external API response time (Graph/OneNote/SharePoint), by Power Automate's own per-action overhead, or by the Topic's own processing between flow calls.
- Whether latency is consistent run-to-run, or whether certain scenarios (e.g. large calendar days, first-ever section creation, recurring vs one-off) are disproportionately slow.
- Whether the P/N/date re-fetch pattern is actually a meaningful contributor to perceived slowness, or a non-issue in practice — this is currently a hypothesis, not a finding.

---

## What to collect this week (David — live usage)

For each real capture during the week, note (Activity trace already records most of this — screenshot or note down):

1. **Total run duration** for the Topic-level conversation, start to finish (roughly: time from first message sent to final "saved to OneNote" confirmation received).
2. **Per-action duration breakdown** from Flow A and Flow B's Activity trace for that run — the small `0s` / `0.2s` / `1s` markers next to each action box, same as used throughout the 23 August sessions. Particularly:
   - `FA08_Get_calendar_view_of_events`
   - `FA09B_Filter_ExcludeLeaveAndPeriodEntries`
   - `FA09C_Sort_CandidatesByStartTime`
   - `Get_items` (SharePoint, Flow B)
   - `Filter_Existing_Mapping`
   - `Create_OneNote_Page` / `Update_page_content_Existing_Branch` (whichever branch fires)
   - `Delay_Post_Page_Creation` (should be a stable ~5s baseline — worth confirming it stays that way)
   - `Create_Mapping_Item_Recurring` / `Create_Mapping_Item_OneOff`
3. **Which branch fired** — new capture vs. recapture vs. recurring vs. one-off vs. P/N/date navigation — since these likely have meaningfully different cost profiles and shouldn't be averaged together.
4. **Any runs that felt subjectively slow** in normal use, even if the trace doesn't obviously explain why — a felt-slowness note is itself useful evidence, distinct from the trace numbers.

No special test scenarios needed — this is meant to be gathered from David's actual week of normal use, which is a better dataset than synthetic testing since it reflects real calendar shapes and real usage patterns.

---

## Candidate areas to analyse once data exists (not yet investigated — listed for reference only)

| Candidate | What it would need to show to matter | Notes |
|---|---|---|
| Multi-layer sequential round trips (Topic ↔ Flow A ↔ Flow B) | A large share of total time sitting in the gaps *between* action executions rather than inside any single action | Would need Topic-level timing, which Flow Activity traces alone may not fully capture |
| Repeated `FA08` calls on P/N/date navigation | `FA08` itself being a meaningfully large fraction of a single run's total, AND navigation being a common enough interaction to matter | If `FA08` is fast in practice, caching it buys little |
| The fixed 5s `Delay_Post_Page_Creation` | Only matters if new-page-creation is a large fraction of total captures — recaptures (the more common case, per the flow's own branch structure) don't hit this delay at all | May be a non-issue if most real usage is recapture, not first-time creation |
| Flow A's filter/sort chain (`FA09`→`FA09B`→`FA09C`) | These being non-trivial relative to `FA08`'s own network-call cost — in-memory array operations on a typically small (single-day) array are usually fast, so this may be a low-value target | Worth ruling out early with data rather than assuming |
| SharePoint `Get_items`/`Filter_Existing_Mapping` in Flow B | Whether the mapping list has grown large enough that a full-list fetch + client-side filter becomes slow, vs. a targeted server-side query | Relevant mainly as the mapping list accumulates rows over time |

None of these are recommendations. They're the set of things worth checking once real per-action data exists — some may turn out to be non-issues.

---

## Next session, once data exists

1. Review the week's collected Activity traces together, tabulate per-action durations across several real runs.
2. Identify which candidate(s) above are actually supported by the data — discard the rest rather than optimizing them speculatively.
3. Only then propose any structural change, following the same evidence-first, scratch-flow-tested, Peek-Code-verified method used throughout the 23 August sessions.
4. Treat `Delay_Post_Page_Creation`'s ~5s as a hard floor unless a genuinely different OneNote-consistency-checking approach is found and proven safe — not a target to shave.

---
*Created 23 August 2026. Scoping only — no analysis performed yet. Update this doc once David's week of usage data is available, or rename/supersede with a dated analysis doc once real findings exist (e.g. `analysis-2026-08-XX-performance-findings.md`, following this repo's existing naming convention).*
