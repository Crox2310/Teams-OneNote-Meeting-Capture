# Session Handover — 2026-06-20 (evening, continued further)

## Status at end of session: AGENT CURRENTLY FAILING LIVE — read this first

The live published agent is currently erroring on every test. Most recent error (repeated twice, ~90 seconds apart):

```
Error Message: The flow 'PA - Resolve OneNote Meeting Section - v2 Clean Build'
('ed112c88-b94b-f111-bec6-002248a38052') failed to run with response code
'BadGateway', error code: NotSpecified. Error details: BadGateway
```

This happened immediately after refreshing a stale connection (see below) and clicking "Allow" on a SharePoint/OneNote consent prompt that reappeared on both attempts without resolving. Recommend NOT making further live changes until this is investigated fresh — see "Recommended next steps" at the end.

## Context: this investigation started as a recurrence-routing trace

Following the FA28A/FA28B fix (see `handover-2026-06-20-evening-fa28-recurrence-fix.md`), the plan was to trace Flow B's recurring branch (`Condition IsRecurring` = True) the way the one-off path was traced earlier today, since Flow A now correctly reports `isrecurring: true` but OneNote outcomes still weren't reflecting recurring-specific behavior.

## Finding 1: Flow B's `Condition IsRecurring` branch was being correctly entered, but everything inside it was Skipped

Run trace (Flow B Activity tab, 12:07 PM run) showed:
- `Condition IsRecurring` itself: **Succeeded** (4s, green check) — confirms the condition evaluated and ran.
- Inside the **True** branch: `Compose Input SeriesMasterId` → `Compose Input MeetingTitle` → `Filter Existing Mapping` → `Compose ExistingPageSelfUrl` → `Compose PageDecision` — **all Skipped**, cascading from the very first action in the branch (`Compose Input SeriesMasterId`, error type `ActionBranchingConditionNotSatisfied`).
- Inside the **False** branch (expanded for comparison): all 5 actions **Succeeded** — `FB-F01 — Compose Input MeetingTitle (one-off)` → `Get Sections OneOff` → `Filter OneNote Section OneOff` → `Compose Section Match Count OneOff` → `Condition Section Exists OneOff`.

Conclusion: `Condition IsRecurring` was evaluating to **False** every time, routing every meeting — including confirmed-recurring ones — down the one-off path, regardless of what Flow A reported.

## Finding 2: `Condition IsRecurring`'s expression itself was checked and is NOT obviously wrong

Expression (Designer, code view):
```
toLower(string(triggerBody()?['IsRecurring'])) is equal to true
```

Checked Flow B's trigger ("When an agent calls the flow") parameter list directly — confirmed the trigger genuinely defines a parameter named `IsRecurring` (capital I, capital R), matching what the expression reads. So this is NOT the same casing-mismatch bug as FA28A/FA28B this morning — the key name is correct. The right-hand side `true` not having visible quotes in the Designer field is a possible type-mismatch suspect (boolean literal vs. the string result of `toLower()`) but was not conclusively diagnosed before the investigation moved elsewhere — **worth revisiting**.

## Finding 3 (the real discovery): Flow B's connection was Stale, and both Topic actions that call it were broken

While trying to inspect `C10_Call_FlowB_Create_Page`'s actual input mapping for `IsRecurring`, found the action showing **"Flow not found or is turned off"** in the Topic canvas. Checked `C8B_Call_FlowB_Create_Page` (the single-match equivalent) for comparison — **same warning**, confirming this was systemic, not isolated to one action.

Investigated three different layers before finding the real cause:
1. Power Automate flow Overview (`v2 Clean Build`) — Status: **On**, with a clean 28-day run history showing repeated `Succeeded` runs. Flow itself is healthy.
2. Copilot Studio **Tools** tab (agent level, not Topic level) — both Flow A and Flow B listed as registered tools, both **Enabled: On**. Flow B's Trigger column shows `By agent` (vs. Flow A's `None`) — a different invocation mechanism than the Topic-action "Call a flow" binding.
3. Copilot Studio **Settings → Connection Settings** — found the actual root cause: `PA - Resolve OneNote Meeting Section - v2 Clean Build` showed **Status: Stale** (red icon), while Flow A and the other connected flow both showed **Connected** (green). This connection is used by "3 tools" — almost certainly `C8B`, `C10`, and the Tools-level `By agent` registration, explaining why all three were affected simultaneously.

Clicked "Review" next to the Stale connection to refresh/re-authenticate it. After this, `C8B` and `C10` both displayed their full Power Automate input panels correctly in the Topic canvas (previously hidden behind the "Flow not found" warning) — confirmed `C10`'s `IsRecurring (String)` input is correctly mapped to the Topic's `IsRecurring` variable, which is the binding we'd originally set out to verify.

## Finding 4: Refreshing the connection did NOT fully resolve it — BadGateway errors followed

Re-tested "capture notes for QWE Meeting" via Teams immediately after the connection refresh. Two consecutive attempts both:
- Surfaced an unresolved SharePoint + OneNote (Business) connection consent prompt ("Allow" / "Cancel") that did not appear to clear even after clicking Allow.
- Resulted in the v2 flow failing with `BadGateway` (see error at top of this document).

This suggests the connection refresh from Connection Settings did not fully propagate, or there is a separate, deeper issue with the SharePoint/OneNote connector credentials specifically (as opposed to the flow-to-Topic binding, which does appear fixed).

## Separate, still-unresolved issue found via Topic checker (not investigated further this session)

Topic checker shows **4 active errors** blocking publish:
- 2x "Flow not found or is turned off" on Action nodes (likely `C8B`/`C10` — may now be resolved by the connection refresh above, not re-checked).
- 2x "Variable is being set to an incorrect type. Assigned: String, expected: Unspecified" on Condition nodes — this corresponds to a live, visible error on `C11_Check_OutStatus`'s condition (`OutStatus | unknown is equal to OK`): **"Incompatible type comparison. Type: String, expected: Unspecified."**

This is a real, currently-active bug in the shared success/failure branching logic that sits downstream of both the C8B and C10 paths — it would affect every user journey, not just the recurring path. Not yet diagnosed or fixed. Likely the same class of bug as the `"@true"` vs `true` Condition Should Create Page fix from earlier in this project (string vs. unspecified/boolean type mismatch).

## Status summary

- FA28A/FA28B recurrence detection (Flow A): **Still fixed, unaffected by this session's later findings.**
- Flow B `Condition IsRecurring` expression: **Not conclusively diagnosed.** Trigger parameter name confirmed correct (`IsRecurring`); type-mismatch on the `true` comparison is a remaining suspect, not yet tested in isolation.
- Flow B connection (SharePoint/OneNote, used by C8B/C10/Tools registration): **Was Stale, refreshed via Connection Settings → Review, but agent is now failing with BadGateway on every live test since the refresh.**
- C11_Check_OutStatus type-mismatch error: **Confirmed live and active, not yet fixed.** Blocks clean publish per Topic checker.
- Live agent: **Currently in a worse functional state than at the start of today's session** — was successfully creating (mis-routed) OneNote pages this morning; is now failing outright with BadGateway errors.

## Recommended next steps (in order)

1. **Do not make further live changes until this is investigated fresh.** The agent is mid-incident.
2. First check: go to Power Automate directly and manually run Flow B (`v2 Clean Build`) outside of Teams/Copilot Studio entirely, using "Run flow" / "Test" with sample inputs. This will tell us whether the BadGateway is a Flow B / connector-level problem (would fail here too) or specifically a Topic-to-flow call path problem (would succeed here, fail only via Teams).
3. If Flow B itself fails when run manually, check the SharePoint and OneNote (Business) connection references directly under Connection Settings — specifically whether re-clicking "Allow" on the consent prompts actually persists, or whether there's a separate expired/revoked credential that needs a full re-authentication (sign out / sign back in) rather than a simple Allow click.
4. Once the connection/BadGateway issue is resolved and confirmed stable (e.g. 2-3 consecutive successful manual runs), re-test "capture notes for QWE Meeting" live and check `Condition IsRecurring` again — there's a reasonable chance the True/False routing issue was itself a symptom of the same stale-connection problem, not a separate logic bug, but this needs to be re-verified once the connection is genuinely healthy.
5. Separately, fix `C11_Check_OutStatus`'s type-mismatch condition — open the condition, check how `OutStatus` is typed at the Topic variable level vs. how `'OK'` is being compared, and align the types (likely needs an explicit string-type variable or a corrected comparison expression).
6. Re-run Topic checker after both fixes and confirm all 4 errors clear before attempting to publish again.
