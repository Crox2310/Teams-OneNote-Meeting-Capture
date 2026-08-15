# Bug 9 — NotFound on Update page content (Existing Branch) — ROOT CAUSE CONFIRMED, 15 August 2026

## Status: ROOT CAUSE CONFIRMED. Not yet fixed. Two candidate fixes identified below.

## Context

Discovered while retesting Bug 5 (one-off recapture, previously failing with empty `sectionId` on `Create_Page_OneOff`). Bug 5's original symptom is confirmed resolved (see below), but a new, distinct failure was found and traced to a genuine schema/logic mismatch — not corruption.

## Bug 5 status: confirmed fixed (original symptom)

- Test 1 (fresh `MeetingId: bug5-retest-15aug-2`, novel title, `IsRecurring: false`): routed correctly through `Create_OneNote_Page` (standard path, since no existing page was found) — succeeded in 0.7s. `Create_Page_OneOff` correctly skipped (dependent condition not met).
- Test 2 (same `MeetingId` reused, forcing an existing-mapping match): `Condition Is Genuine Existing Page` correctly evaluated **True** — `varOneNoteResolverResult` correctly resolved to `ExistingSection`. The flow correctly routed to the "update existing page" branch rather than falling through to `Create_Page_OneOff` with an empty `sectionId` — **the original Bug 5 symptom did not reproduce.**

Bug 5's root cause (corrupted `varTargetSectionPagesUrl`/`varOneNoteResolverResult` SetVariable actions, fixed earlier this session) is confirmed resolved.

## Bug 9: `NotFound` on `Update page content Existing Branch`

- **Action**: `Update page content Existing Branch` (inside `Apply to each Existing Section`, inside `Condition Is Genuine Existing Page` → True branch)
- **Error**: `NotFound`
- **Test 2 timestamp**: 15 August 2026, ~17:47

### Investigation trail

1. **First theory (WRONG, since corrected)**: suspected the SharePoint mapping row was never written by `OF09a`, based on an initial (stale, unrefreshed) view of the `RecurringMeetingSectionMap` list that appeared to show only one row.
2. **Corrected after refreshing the list view**: the mapping row from Test 1 *does* exist — `OF09a — Send an HTTP request to SharePoint (OneOff)` genuinely succeeded (`statusCode: 201`, `Id: 109`, confirmed via that run's raw outputs). The row shows `MeetingTitle: Mtg - Bug 5 Retest 15 Aug`, `MeetingId: bug5-retest-15aug-2`, `Status: Active`, and populated `PageSelfUrl`/`PageWebUrl` values. So the insert itself worked.
3. **Root cause found**: this row is flagged in SharePoint's UI with a "Required info" banner and appears under "Items that need attention." Opening the item confirms: **`SeriesMasterId` is a required field (red asterisk) in the list schema, and this row has it blank** — shown as "Enter value here" in red.

### Confirmed mechanism

- The `RecurringMeetingSectionMap` SharePoint list has `SeriesMasterId` configured as a **required column**.
- The **one-off branch's** insert (`OF09a — Send an HTTP request to SharePoint (OneOff)`) never includes `SeriesMasterId` in its payload — by design, since one-off meetings have no series master. Its body only sends `Title`, `MeetingId`, `MeetingTitle`, `SectionPagesUrl`, `Status`.
- SharePoint's REST API accepted the insert anyway (`201 Created`), but the resulting item is left in an incomplete/flagged state in the SharePoint UI (visible as "needs attention").
- This is a genuine **schema/logic parity gap** between the recurring branch (which always populates `SeriesMasterId`) and the one-off branch (which structurally cannot, since the field doesn't apply) — not a corruption artifact, and not the same mechanism as Bugs 4–8 documented elsewhere this session.
- Working hypothesis for *why* this produces `NotFound` downstream: some SharePoint connector operations, filtered views, or list-level validation behave inconsistently with items sitting in an incomplete-required-field state, which may affect how a later query or lookup against this row resolves. **This causal link (incomplete row → NotFound on OneNote update) has not been definitively proven — it's the most likely explanation given the timing and the fact this is the only distinguishing anomaly found on the item, but it hasn't been isolated by removing the flagged state and re-testing.**

### Candidate fixes (not yet applied)

1. **Make `SeriesMasterId` optional in the SharePoint list schema.** Simplest fix — the field is genuinely inapplicable to one-off meetings, so removing the required constraint reflects the actual data model. Risk: need to confirm no other part of the flow or any other consumer of this list relies on `SeriesMasterId` always being populated.
2. **Have `OF09a`'s insert send a placeholder value for `SeriesMasterId`** (e.g., reuse `MeetingId`, or a literal sentinel like `"N/A"` or `"one-off"`) to satisfy the required-field constraint without changing the schema. Lower risk to other consumers of the list, but adds a slightly awkward placeholder value that needs documenting so it isn't mistaken for a real series master ID later.

**Recommendation**: option 1 (schema change) is cleaner long-term, since it fixes the actual mismatch rather than working around it. Confirm with whoever owns the SharePoint list/downstream reporting (if any) before changing the schema, in case something else depends on the field being mandatory.

## Next steps

1. Decide between the two candidate fixes above (schema change vs. placeholder value).
2. Apply the chosen fix, then re-run the same two-step Bug 5/9 test sequence (fresh one-off capture, then a second run reusing the same `MeetingId`) to confirm `Update page content Existing Branch` now succeeds instead of `NotFound`.
3. Check whether the *recurring* branch's row created via the Bug 8 test earlier today (also missing `PageSelfUrl` initially, now populated) has any similar "needs attention" flag — if so, this may point to a second required field being inconsistently populated, worth a quick scan of the full schema's required-field list against what each branch's insert actually sends.

---

*Logged 15 August 2026. Root cause confirmed same session after an initial incorrect theory (stale list view) was corrected by David spotting the "Required info" flag directly. See `handover-2026-08-15-session2-part2-recovery-complete-published.md` for full session context.*
