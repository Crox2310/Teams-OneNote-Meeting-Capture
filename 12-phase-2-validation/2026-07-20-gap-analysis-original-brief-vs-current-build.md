# Gap Analysis — Original Brief vs Current Build

**Date:** 2026-07-20
**Scope:** Full repository review against the original project brief and expectations (`00-overview`, `01-shared-contract`, `02-user-journeys`, `06-decisions`, `08-build-checklists`, `11-baseline`), cross-referenced against the current live implementation and the now-complete UJ1–UJ5 validation records in `12-phase-2-validation/`.

**Trigger:** All five original user journeys passed live validation for the first time in the project's history on 2026-07-19/20. This review asks a broader question than "do UJ1–5 pass" — it asks what the original brief specified that is still not built, now that the meeting-selection design has evolved beyond the original spec.

---

## 1. Summary

The core OneNote/SharePoint mechanics for all five journeys are now built and validated for the first time. The gaps identified here are not "the feature is broken" — they are pieces of the original brief that were never built at all, or that the build simplified away during construction.

| Area | Status |
|---|---|
| UJ1 — one-off single match | Matches original spec, validated |
| UJ2 — multiple match selection | Matches original spec, validated |
| UJ3 — recurring, existing mapping | Core happy path matches spec, validated. Stale-row/duplicate-row handling not built |
| UJ4 — recurring, first-time setup | Core happy path matches spec, validated. User choice, blank-key fallback, retry control not built |
| UJ5 — no-match recovery | Core no-match detection matches spec, validated. Original recovery menu not built — superseded by an approved design improvement (P/N/date-jump), which itself goes beyond original scope |
| Flow B `OutStatus` differentiation | Not built — single hardcoded value | 
| Amendment log / change control | Process defined but never used — no amendments logged despite significant fixes this week |

---

## 2. Navigation design change (approved)

**Original spec (UJ5):** on no-match, offer a menu — "1. Search today's meetings again with different wording, 2. Stop."

**What was actually built:** P/N (previous/next day) and free-text date-jump navigation, layered on top of the existing day-query mechanism. This lets a user page across days at any point in the conversation, not only after a no-match. It has been validated (see `2026-07-20-date-jump-feature-and-uj-validation.md`, `uj5-validation-record.md`).

**Decision:** confirmed as an improvement over the original design. Agreed next step is to build out the remaining piece of the original idea that P/N/date-jump does not cover — a same-day reword/retry option — as an addition alongside the existing navigation, not a replacement for it. UJ5's current recovery message should be extended to offer this alongside P/N/date, and an explicit "Stop" exit should be added so the conversation can be deliberately ended rather than only navigated away from.

**Action:** treat as a controlled amendment (see Section 5) — update `02-user-journeys/user-journey-5-no-match-recovery.md` to reflect the combined design once built, rather than leaving the original menu as the on-paper spec while a different behaviour ships.

---

## 3. Flow B `OutStatus` differentiation — not built

**Spec (`01-shared-contract/shared-journey-contract-vfinal.md`):** Flow B should return one of six distinct `OutStatus` values — `SUCCESS`, `RECURRING_SETUP_REQUIRED`, `PARTIAL_SUCCESS`, `SETUP_SECTION_NOT_FOUND`, `SETUP_SECTION_AMBIGUOUS`, `ERROR` — and the Topic routes on it.

**Actual build:** `Set_varOutStatus` is hardcoded to `"OK"` at a single point in the flow. No other value is ever produced. This was also the source of a regression fixed this week (empty string instead of `"OK"`, causing false "something went wrong" errors despite success) — the underlying fragility (one hardcoded value, no real branching) is what allowed that regression to happen invisibly.

**Why this can't be a Topic-only fix:** the Topic can only route on what Flow B actually sends. Building Topic-side conditions before Flow B differentiates the status would produce dead code with nothing real to trigger it — the same failure mode as `Condition_IsRecurring` sitting unreachable for the project's entire history until fixed this week.

**What's required, in order:**

1. **Flow B** — add explicit status-setting at each branch point that already exists in the flow:
   - `SUCCESS` after a clean create/update completes.
   - `RECURRING_SETUP_REQUIRED` when `Condition Mapping Exists` is False with a stale or invalid existing row (currently the flow only distinguishes "row exists" vs "doesn't" — see Section 4).
   - `SETUP_SECTION_NOT_FOUND` / `SETUP_SECTION_AMBIGUOUS` from the section-match-count logic in `Condition Section Exists Recurring`.
   - `PARTIAL_SUCCESS` if the OneNote page write succeeds but the SharePoint mapping write fails.
   - `ERROR` as a genuine catch-all — this requires adding "configure run after" failure handling to the OneNote and SharePoint actions, which Flow B does not currently have. At present a failed action fails the whole flow rather than surfacing a status.
2. **Topic** — once Flow B emits real values, add conditions that route on `OutStatus`:
   - Re-prompt once on `SETUP_SECTION_NOT_FOUND` / `SETUP_SECTION_AMBIGUOUS`, governed by `Topic.SectionRetryCount` (see Section 4).
   - Distinct user-facing message for `PARTIAL_SUCCESS`.
   - `RECURRING_SETUP_REQUIRED` loops back into the UJ4 setup path.
   - Safe, generic message for `ERROR`.

This is the single highest-leverage item on this list — several of the gaps below are blocked on it.

---

## 4. UJ4 gaps — first-time recurring setup

Three pieces of the original spec were never built:

**User choice of section.** Original spec: the user is offered "1. Use an existing OneNote section, 2. Create a new OneNote section." What's built instead is fully automatic — the flow matches by name and silently reuses or creates, with no user-facing choice. Functionally reliable, but removes the user's ability to deliberately pick a different existing section than the auto-matched one.

**Blank `SeriesMasterId` fallback.** Original spec: if a recurring meeting has no usable `SeriesMasterId`, do not create a mapping — instead offer "1. Capture as a one-off note now, 2. Skip for now." Not built. This connects to a previously identified, still-open gap in Flow A (`FA43`, from the 2026-07-16 root-cause doc) where `IsRecurring`/`SeriesMasterId` are not fully coalesced, meaning this edge case could still reach Flow B unhandled.

**`Topic.SectionRetryCount`.** Original spec: initialise to `0`, cap at `1`, then graceful exit — used to bound the re-prompt loop on ambiguous/not-found section lookups. Not built, because it has nothing to attach to until `OutStatus` differentiation (Section 3) exists.

---

## 5. UJ3 gaps — recurring, existing mapping

**Spec:** `0 rows → RECURRING_SETUP_REQUIRED → UJ4`, `1 accessible row → append`, `1 stale row → RECURRING_SETUP_REQUIRED → UJ4`, `2+ rows → ERROR`.

**Built:** `Condition Mapping Exists` is a simple boolean — a row exists or it doesn't. There is no check for a *stale* single row (e.g. pointing at a deleted OneNote section) falling back to setup, and no detection of a duplicate/multi-row state producing `ERROR`. In practice, a stale row would currently be treated as valid and likely fail later at the OneNote write step rather than being caught upfront and routed to UJ4 gracefully.

---

## 6. Process debt — controlled amendment process not in use

`11-baseline/final-completion-note.md` and `11-baseline/amendment-log.md` establish a mandatory rule: "No ad hoc patching is permitted outside this process." `amendment-log.md` as it stands on GitHub is still the unfilled template — no amendment has ever been logged.

Since the last baseline checkpoint, the following fixes have been applied directly without going through the logged amendment process: the P/N date-navigation fixes (Topic + Flow A), the `Condition_IsRecurring` string/boolean type bug, the `Condition_Should_Write_Mapping` duplicate-condition bug, the missing section-creation chain, the `Compose_SafeSectionName` naming mismatch, the `Set_varOutStatus` empty-string regression, and the date-jump feature build.

None of these were wrong to fix — several were blocking bugs — but per the project's own rules they should each have a logged `AMEND-YYYY-MM-DD-NNN` entry recording root cause, affected journey, affected baseline files, and the design/build/test correction applied. `living-audit.md` is also still missing the individual per-action entries for this week's work.

**Recommendation:** before starting the `OutStatus` build (Section 3), backfill amendment-log entries for this week's fixes, then log the `OutStatus` work itself as a new amendment from the start rather than another ad hoc patch.

---

## 7. Confirmed matching original spec (no gap)

For completeness — these were checked against the original brief and found to match:

- UJ1's entry conditions, behaviour, and guardrails (no recurring questions, no mapping writes, Flow B call gate) — matches `02-user-journeys/user-journey-1-one-off-single-match.md` exactly.
- UJ2's core mechanism — first Flow A call with blank `InSelectedNumber`, second call with the user's numeric selection — matches `02-user-journeys/user-journey-2-multiple-meetings-selection.md` exactly. This is unaffected by the P/N/date-jump work, which operates at the day-query level, not the candidate-selection level.
- The Universal Flow B call gate (`FlowAStatus = SINGLE_MATCH` AND `MeetingTitle`/`CalendarEventId`/`PageHtml` not empty) from `01-shared-contract/shared-journey-contract-vfinal.md` is respected.
- Outlook Data Capture Profile V1 — no attachment binary content, inclusion flags as text `true`/`false` — matches `01-shared-contract/outlook-meeting-data-capture-profile-v1.md`.
- `SeriesMasterId` treated as an opaque key (Decision Log vFinal, item 5) — matches.

---

## 8. Recommended build order

```text
1. Backfill amendment-log.md entries for this week's fixes (process compliance, low effort)
2. Flow B OutStatus differentiation + error-branch handling (unblocks everything below)
3. Topic-side OutStatus routing + Topic.SectionRetryCount
4. UJ3 stale-row / 2+ row handling (depends on 2)
5. UJ4 blank-SeriesMasterId one-off fallback (depends on FA43 fix in Flow A)
6. UJ4 user choice of section (independent — can be built any time)
7. UJ5 reword/retry + explicit Stop option, alongside existing P/N/date-jump
8. Update 02-user-journeys/user-journey-5-no-match-recovery.md to reflect the combined final design
```

Items 2–5 are the only ones with a hard dependency chain. Items 6–8 can be built in any order once bandwidth allows.
