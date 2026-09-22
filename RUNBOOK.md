# Runbook — Meeting Capture

Checks, recovery steps and troubleshooting for the live system. The design is in `ARCHITECTURE.md`; open issues are in `BACKLOG.md`.

---

## 1. Before changing anything

Do these at the start of every build or fix session.

1. Open Flows A, B, C and D and run Flow Checker on each. Also run the Topic checker in Copilot Studio.
2. Wait 20–30 seconds after opening a flow before trusting what you see. Fields can show as empty while the Designer is still loading.
3. In Code view, spot-check the key SetVariable values against the known-good references:
   - Flow A: `known-good-values-flow-a-reference.md`
   - Flow B: `known-good-values-master-reference.md` and `known-good-values-addendum-2026-09-18.md`
   - Flows C and D: the latest `flow-reference-*` snapshots
4. Confirm Express mode is off.
5. Snapshot any flow you're about to change (Code view → GitHub).

Flow Checker showing 0 errors doesn't prove a flow is healthy. The present-but-empty value corruption doesn't show up in Flow Checker at all.

---

## 2. Making a change safely

1. Prefer editing an expression over adding, moving or deleting actions. Structural edits are what set off the value wipe.
2. After each structural edit: save → close → reopen → wait → check Code view → publish.
3. Always publish before testing. The agent and Flow D call the published version, not your draft.
4. Test from a new Teams chat with the agent.
5. Test new expressions in **PA - Scratch Diagnostics** first.
6. Power Automate connector actions have to be built in the Designer UI. Pasting code doesn't work for them.

---

## 3. Recovering from a value wipe

**Symptoms:** "Value is required" errors in Flow Checker, `"value": ""` or a missing `value` key in Code view, or a run failing on a variable that used to work.

1. Stop editing that flow.
2. List every affected action from Flow Checker, then check Code view for present-but-empty values that Flow Checker missed.
3. Restore each value from the matching known-good reference. Don't restore actions that were added after the reference was written; check those by hand.
4. Save → close → reopen → check again. Wipes can come back during the same session.
5. If the flow keeps wiping, restore a known-good version from **Version History** instead.
6. Publish, run one known-good test, and log the incident with the date and the actions affected.

---

## 4. A page didn't fill (no recap, no chat)

Work through these in order.

1. **Did Flow D see the row?** Open Flow D's run history for a run after the meeting ended plus 30 minutes. Is the row in FD04 or FD04b's output?
   - It isn't there: check the row's `OccurrenceDate` (must be today or yesterday in `yyyy-MM-dd`), `ChatCaptured` (must be No), and `EndTime` (must not be empty). A one-off row also needs a `MeetingId`, and a recurring row needs a `SeriesMasterId`.
2. **Did FD06c decide the time was up?** Compare `utcNow()` with `EndTime` plus the offset. `EndTime` should be UTC in ISO format. A US-style value such as `09/22/2026 09:30:00` may be misread (BL-01).
3. **Did FD06d run?** If it was skipped even though FD06c took the True branch, that's BL-01.
4. **Did Flow C fail?** Open the child run:
   - Failed at FC05 with "Lookup value is required": the row has no `JoinUrl` (see BL-02 for one-offs).
   - Failed at FC05g: check that `aiInsightId` reads `@outputs('FC05f_Best_Insight_Id')`, including the `@`.
   - An expression that reads connector output returns null although the run viewer shows data: check the `?['body']` path segment. Some connectors unwrap the HTTP envelope.
5. **Did Flow C succeed but the page looks unchanged?** Check FC05u or FC22 to see which row it updated, and which page id FC14 and FC05t wrote to.

## 5. Page has chat but no recap

1. Check whether Teams actually produced a recap for that meeting (open the meeting's Recap tab in Teams).
2. Check the Flow D offset (BL-04). If it's too short, Flow C runs before the recap exists and then marks the row done.
3. To try again, force a re-capture (§6). This adds a second chat block.

---

## 6. Forcing a re-capture

1. In `RecurringMeetingSectionMap`, set the row's `ChatCaptured` to No.
2. For an occurrence older than yesterday, temporarily set `OccurrenceDate` to yesterday's date so Flow D picks it up. Put the real date back afterwards.
3. Make sure `EndTime` is a UTC ISO value.
4. Wait for the next Flow D poll (every 10 minutes), or run Flow C manually with SeriesMasterId, MeetingTitle and OccurrenceDate (plus MeetingId for a one-off).

Writes are append-only, so the page will show a second recap and chat block. Delete the duplicate by hand in OneNote.

---

## 7. Agent says "something went wrong" but the page exists

Flow B completed but `OutStatus` wasn't `SUCCESS`.

1. Open the Flow B run and check `varPageAction` and `varOutStatus`.
2. If `varOutStatus` is empty, the Set_varOutStatus value has been wiped. Restore it (§3).
3. Check the mapping row was written. If the title-set step failed, the mapping write is skipped (BL-08), and re-running creates a duplicate page.

## 8. Agent errors before listing meetings, or a flow can't be found

- **"Flow not found or is turned off", or the Topic checker shows errors on C2 or C10:** the tool link in Copilot Studio is broken. This happens whenever a flow's trigger inputs change. Re-link the flow, check the Topic checker is clean, then publish the agent.
- **ContentValidationError:** a URL or other special characters have been passed as a separate flow input. The join URL must travel inside `text_3` (see `ARCHITECTURE.md` §3.1).

---

## 9. Regular housekeeping

- **Mapping list size.** Watch Flow B's `OutSPItemCount`. When the list nears 350 rows, prioritise BL-06.
- **Stale rows.** Remove rows left by failed Flow B runs (an empty `SectionPagesUrl`) until BL-07 is fixed.
- **Snapshots.** After any published change, snapshot the flow in Code view to `12-phase-2-validation/` and update the known-good references.
