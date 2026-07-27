# Flow B — `Condition Is Genuine Existing Page` Structurally Unreachable

**Date:** 2026-07-27
**Method:** Live test of AMEND-2026-07-27-001's fixes (re-capturing a meeting with an existing page), followed by a targeted Peek Code check of `Set_varOneNoteResolverResult_Exists_OneOff` after the live run showed `Condition Is Genuine Existing Page` evaluating False. Cross-checked against every other `varOneNoteResolverResult`-setting action already captured in `2026-07-27-flow-b-outstatus-trace.md`.
**Status:** Root cause confirmed, and real-world impact now confirmed live (see Section 3a) — this defect is actively creating duplicate OneNote pages. Not yet fixed — write-up only, pending a design decision (see Section 4).

---

## 1. Background

While live-testing AMEND-2026-07-27-001's fixes (`varPageAction`/`varOutputPageSelfUrl` corrections on the "page already exists" branch), the test run showed `Condition Is Genuine Existing Page` — which gates whether Flow B appends an "automated update" note to a genuinely existing OneNote page, versus falling back to a one-off page creation — evaluating **False**.

Rather than assume this was simply the wrong test scenario, a second capture of the same meeting was considered as a way to hit the True branch. Before doing so, the actual literal value the condition checks for was investigated directly via Peek Code, since a repeat capture would only be a useful test if the True branch was actually reachable at all.

## 2. Finding

`Condition Is Genuine Existing Page`'s expression:
```
equals(variables('varOneNoteResolverResult'), 'Exists')
```

`Set_varOneNoteResolverResult_Exists_OneOff` — the action inside `Condition Section Exists OneOff`'s True branch, named as though it sets `varOneNoteResolverResult` to `"Exists"` — actually contains no `value` key at all:
```json
{
  "type": "SetVariable",
  "inputs": {
    "name": "varOneNoteResolverResult"
  },
  "runAfter": {
    "Set_varTargetSectionPagesUrl_OneOff_Exists": ["Succeeded"]
  }
}
```

This is not a wrong-value bug — it assigns nothing at all, leaving `varOneNoteResolverResult` at whatever it was previously (most likely blank/uninitialized on this path, since Flow B's variable-init block at the top of the flow was not seen to include it).

Cross-checking against every other action anywhere in Flow B that sets `varOneNoteResolverResult` (all previously captured in `2026-07-27-flow-b-outstatus-trace.md`, Section 2):

| Action | Path | Value set |
|---|---|---|
| `varOneNoteResolverResult_ExistingMapping` | Recurring, mapping row exists | `"ExistingMapping"` |
| `varOneNoteResolverResult_1` | Recurring, section exists | `"ExistingSection"` |
| `varOneNoteResolverResult_2` | Recurring, section created | `"CreatedSection"` |
| `Set_varOneNoteResolverResult_Exists_OneOff` | One-off, section exists | *(no value assigned — see above)* |
| *(one-off, section created)* | Not yet traced this session — see Section 3 |  |

**None of the confirmed values is the literal string `"Exists"`.** This means `Condition Is Genuine Existing Page` cannot evaluate True via any currently-traced code path in Flow B. It is structurally unreachable — the same failure pattern previously found and fixed in `Condition_IsRecurring` (AMEND-2026-07-19-001) and `C6B_Check_N` (AMEND-2026-07-18-001): a condition that reads correctly on the canvas but has no upstream action that can ever satisfy it.

## 3. Downstream consequence

If this condition can never be True, then Flow B's False branch — `Create_Page_OneOff`, a fresh page creation — fires every time the "page already exists" route is taken, **regardless of whether a genuine existing page was actually found**. In practice this likely means:

- The "append an automated update note to an existing page, preserving human-entered notes" feature (`Compose_UpdateHtmlFragment`, `Update_page_content_Existing_Branch`) has probably never executed for any meeting, on any path, at any point in the project's history.
- Re-capturing a meeting whose section/page already exists may be creating a **new** OneNote page each time instead of updating the existing one — worth confirming directly, since this would mean duplicate pages accumulating silently, similar in spirit to the duplicate-section defect fixed in AMEND-2026-07-19-004.

This has not yet been confirmed by checking actual OneNote content (e.g. whether repeated captures of the same meeting have produced multiple pages) — recommended as the first verification step before deciding on a fix.

## 3a. Real-world impact confirmed live

To confirm, "HoP - Focus Time" (a recurring meeting, section "Mtg - HoP - Focus Time") was deliberately re-captured for the same occurrence (27 Jul 2026) it had already been captured for earlier in the session. The agent responded with a normal-looking success message and a page link, giving no indication anything was wrong.

Checking the OneNote section directly afterward showed **two separate pages both dated "27 Jul 2026"** under "Mtg - HoP - Focus Time" (alongside the unrelated, correctly-separate "3 Aug 2026" occurrence). This confirms the predicted consequence exactly: the recapture did not update the existing 27 Jul page, it silently created a duplicate.

This confirms the defect is real-world impacting today, not merely theoretical, and affects the recurring path as well as the one-off path (this test used a recurring meeting) — meaning the recurring-side equivalents of `varOneNoteResolverResult` (`"ExistingMapping"`, `"ExistingSection"`, `"CreatedSection"`) are equally unable to satisfy `Condition Is Genuine Existing Page`, consistent with Section 2's table.

## 4. Not yet resolved

- The one-off "section created" branch's equivalent action was not traced this session — worth checking whether it has the same missing-value defect or a different (but still non-`"Exists"`) value.
- **Design decision needed**: should the fix restore each of these five (or more) actions to set `"Exists"` literally wherever appropriate, or should `Condition Is Genuine Existing Page`'s expression be changed to match the values actually in use (e.g. `contains(createArray('ExistingMapping','ExistingSection','ExistsOneOff'), variables('varOneNoteResolverResult'))`)? The former is simpler; the latter avoids collapsing genuinely distinct states (mapping-existed vs section-existed vs section-was-created) into one word, which may itself be useful information for the `OutStatus` work now being planned.
- Live confirmation needed: check actual OneNote content for a repeatedly-captured meeting to see whether duplicate pages have been created as a result of this defect.

## 5. Relationship to the `OutStatus` build

This defect was found via the same trace exercise being done ahead of the `OutStatus` six-value build (`2026-07-27-flow-b-outstatus-trace.md`). It's relevant to that work directly: `varOneNoteResolverResult`'s distinct states (`ExistingMapping`/`ExistingSection`/`CreatedSection`/one-off equivalents) look like good candidate inputs for distinguishing `SUCCESS` from `PARTIAL_SUCCESS`, or for informing response messaging about whether a page was created vs updated — but only once they reliably and correctly reach `Condition Is Genuine Existing Page`, or that condition is redesigned around the values that actually exist.

## 6. Suggested next steps

1. ~~Check actual OneNote content for evidence of duplicate pages from repeated captures of the same meeting~~ — **done, confirmed, see Section 3a.**
2. Trace the untraced one-off "section created" branch to complete the picture.
3. Make the design decision in Section 4 (restore `"Exists"` literally, vs redesign the condition around real values).
4. Fix and live-test, following the controlled amendment process. Given confirmed real-world impact (silent duplicate page creation on every recapture of an existing meeting, recurring or one-off), this should be treated as a priority fix rather than deferred alongside the `OutStatus` build.
5. Consider a one-off cleanup pass to identify and remove any other duplicate pages already created by this defect across the project's history, since it appears to have been present since this branch was originally built.
6. Log as a new amendment once fixed — this document is a trace/investigation record only.
