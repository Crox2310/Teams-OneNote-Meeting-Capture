# Amendment Log

## Purpose

This log records controlled amendments to the Teams-OneNote-Meeting-Capture baseline.

No ad hoc patching is permitted outside this process.

## Amendment process

When an issue or new learning is found:

1. Stop the build or test.
2. Capture the exact failure.
3. Diagnose root cause.
4. Identify affected layer.
5. Identify affected baseline files.
6. Define the design correction.
7. Define the build correction.
8. Define the test correction.
9. Update GitHub source of truth.
10. Mirror the update to SharePoint Knowledge.
11. Apply the implementation change.
12. Re-test affected journeys.

## Amendment template

### Amendment ID

AMEND-YYYY-MM-DD-001

### Amendment title

Short description.

### Reason for amendment

What failed or what new connector learning was discovered?

### Affected area

- Flow A
- Meeting Capture topic
- Flow B
- SharePoint
- OneNote
- Test matrix
- Build Coach
- Documentation

### Affected user journey

- UJ1
- UJ2
- UJ3
- UJ4
- UJ5
- Regression

### Affected baseline files

List exact files to update.

### Required design correction

Describe the design change.

### Required build correction

Describe the implementation change.

### Required test update

Describe the test that must be added or amended.

### SharePoint mirror update required

Yes / No

### Status

Proposed / Approved / Applied / Retested / Closed

---

## Logged amendments — backfilled 2026-07-20

The amendments below were applied ad hoc during live debugging between 2026-07-18 and 2026-07-20, ahead of being logged through this process. They are backfilled here in full to bring the baseline record into line with the controlled amendment rule, and to correct `baseline-register.md`, which had declared a v1.0.0 "GO" production-readiness decision before these defects were found. See the corresponding entry in `baseline-register.md` for the resulting version correction.

---

### Amendment ID

AMEND-2026-07-18-001

### Amendment title

Day-navigation (P/N) date resolution defects

### Reason for amendment

P/N day-paging appeared to work in conversation but did not actually shift the query date. Two independent defects were found: (1) the Topic's date-shift expressions passed a text date directly into `DateAdd()`, which requires a date value, causing a parse error; (2) Flow A's `FA04_Init_varDateContext` read `triggerBody()?['DateContext']`, but the trigger's actual JSON schema property name is `text_3` — `"DateContext"` was only the field's display label, not its key. The flow silently read a blank/wrong value every time.

### Affected area

- Meeting Capture topic
- Flow A

### Affected user journey

- Regression (cross-cutting — blocks reliable entry into UJ1/UJ2/UJ3/UJ4 from any day other than the default)

### Affected baseline files

- `04-agent-topic-flow-map/` (Topic date-context logic)
- Flow A action `FA04_Init_varDateContext`

### Required design correction

Topic date-shift expressions must wrap the stored date string in `DateValue()` before passing it to `DateAdd()`, and re-wrap the result in `Text(..., "yyyy-MM-dd")` before storing it back to `Topic.DateContext`. Flow A must read the trigger's real schema key, not its display label.

### Required build correction

- `C1_Set_DateContext`: `=Text(Today(), "yyyy-MM-dd")`
- P/N `SetVariable` actions: `=Text(DateAdd(DateValue(Topic.DateContext), -1, TimeUnit.Days), "yyyy-MM-dd")` (P), `+1` (N)
- `FA04_Init_varDateContext`: `triggerBody()?['text_3']`

### Required test update

Live P/N navigation test confirming the queried day actually shifts, verified via Flow A's `FA06_Compose_StartOfDayUtc` Activity trace output, not just absence of a crash.

### SharePoint mirror update required

No

### Status

Retested — confirmed working live 2026-07-18, see `2026-07-18-pn-navigation-topic-fixes.md`.

---

### Amendment ID

AMEND-2026-07-18-002

### Amendment title

Topic Checker type-inference errors on Flow A output variables

### Reason for amendment

Topic Checker reported 7 errors, 5 of them "Unspecified"/unknown-type errors on `MatchCount`, `IsRecurring`, `MeetingTitle`, `SeriesMasterId`, and `CalendarEventId`. Variables populated via `InvokeFlowAction` output bindings default to an unresolved type in Copilot Studio's own type system regardless of the flow's declared schema types. Condition actions' left-hand side in this build only supports a raw variable picker with no formula/fx editor, so the type could not be fixed at the point of use.

### Affected area

- Meeting Capture topic

### Affected user journey

- Regression (build-quality; blocks clean Publish and Checker validation)

### Affected baseline files

- `04-agent-topic-flow-map/`

### Required design correction

None — this is a Copilot Studio type-inference limitation, not a design defect.

### Required build correction

Reassign each affected variable to itself through a `Text()`-wrapped `SetVariable` step immediately after the `InvokeFlowAction` call, forcing correct type inference.

### Required test update

Confirm Topic Checker error count drops from 7 to 2 (the 2 remaining are the cosmetic "Flow not found or is turned off" warnings covered below) and Publish succeeds with a green banner.

### SharePoint mirror update required

No

### Status

Retested — confirmed 2026-07-18. The 2 remaining "Flow not found" Checker warnings were confirmed cosmetic (Publish succeeds, live testing works) and require no correction.

---

### Amendment ID

AMEND-2026-07-19-001

### Amendment title

`Condition_IsRecurring` string/boolean type-coercion defect — recurring path never executed

### Reason for amendment

`Condition_IsRecurring` in Flow B never evaluated True for any meeting, for the entire life of the project. Root cause: typing a plain literal (`true`) into a Condition action's simple "value equals value" UI box, without using the `fx` expression editor, causes Power Automate Designer to silently infer and store it as a native JSON boolean, which will never string-equal a `string(...)`-wrapped comparison value on the other side — even though both display identically as "true" in the UI. This meant every recurring meeting silently fell through to the one-off code path instead of the recurring-mapping path, for the entire project history. All prior "successful" recurring-meeting tests had been false positives.

### Affected area

- Flow B

### Affected user journey

- UJ3
- UJ4

### Affected baseline files

- `08-build-checklists/flow-b-connector-validation-gates.md`
- `12-phase-2-validation/` (supersedes any prior recurring-path test results recorded before this fix)

### Required design correction

Condition comparisons involving a boolean-looking literal must be written as a single self-contained expression on one side (typically the left, via the `fx` editor), e.g. `equals(toLower(string(triggerBody()?['text'])), 'true')`, rather than splitting the expression and a typed literal across the builder's two boxes.

### Required build correction

Rewrote the entire left-hand box of `Condition_IsRecurring` as the full self-contained expression above, leaving the right-hand `true` chip (entered via `fx`, not typed) unchanged. As a direct downstream consequence of this branch executing for the first time, `Set varFinalMatchCount_1` also surfaced a String/Integer type mismatch, fixed by wrapping its value in `string(...)`.

### Required test update

Live recurring-meeting capture confirming `Condition_IsRecurring` evaluates True via the Flow B Activity trace (explicit True/False tags), not inferred from downstream success alone.

### SharePoint mirror update required

Yes — this defect and its root-cause pattern (Power Automate Condition-builder type coercion) should be added to connector learnings.

### Status

Retested — confirmed 2026-07-19, see `2026-07-18-flow-b-mapping-exists-trace.md` and live UJ3/UJ4 validation records.

---

### Amendment ID

AMEND-2026-07-19-002

### Amendment title

`Condition_Should_Write_Mapping` duplicated its parent's condition — SharePoint mapping-write unreachable

### Reason for amendment

`Condition_Should_Write_Mapping`'s expression (`greater(int(...varFinalMatchCount...), 0) == true`) was an exact duplicate of its parent condition's already-false check, making the branch that writes a new `RecurringMeetingSectionMap` row for a new recurring series structurally unreachable regardless of the `Condition_IsRecurring` fix above.

### Affected area

- Flow B
- SharePoint

### Affected user journey

- UJ4

### Affected baseline files

- `01-shared-contract/shared-journey-contract-vfinal.md` (no field-level change required, logic-only defect)

### Required design correction

The mapping-write gate should test `IsRecurring`, not re-test the parent's match-count condition.

### Required build correction

Replaced the left-hand expression with `equals(toLower(string(triggerBody()?['text'])), 'true')`, keeping the right side `true` unchanged.

### Required test update

Live test confirming a new SharePoint mapping row is written (HTTP 201) for a genuinely new recurring series.

### SharePoint mirror update required

No

### Status

Retested — confirmed 2026-07-18/19, live write returned SharePoint `Id: 20`, `statusCode: 201`.

---

### Amendment ID

AMEND-2026-07-19-003

### Amendment title

Missing OneNote section-creation logic for first-time recurring series

### Reason for amendment

No logic existed anywhere in Flow B to create a new OneNote section when a recurring meeting had no existing SharePoint mapping. `varTargetSectionPagesUrl` was never populated for a genuinely new series, causing `Create OneNote Page` to fail with "The section id given in the input is invalid." This was invisible until AMEND-2026-07-19-001 was fixed, since the recurring branch had never executed before.

### Affected area

- Flow B
- OneNote

### Affected user journey

- UJ4

### Affected baseline files

- `08-build-checklists/flow-b-connector-validation-gates.md`
- `04-agent-topic-flow-map/`

### Required design correction

UJ4's "resolve or create the OneNote section" behaviour (per `02-user-journeys/user-journey-4-first-time-recurring-setup.md`) needs a real action chain mirroring the existing one-off section-resolution logic.

### Required build correction

Built a new chain inside `Condition Mapping Exists`'s False branch: `Get Sections Recurring` → `Filter OneNote Section Recurring` → `Compose Section Match Count Recurring` → `Condition Section Exists Recurring`, with a True branch reusing an existing section by name match and a False branch creating one via "Create a section" (not "Create page in a section").

### Required test update

Live test on a genuinely new recurring series confirming a correctly-named section is created and the page write succeeds.

### SharePoint mirror update required

Yes — note that UJ4's "existing section vs new section" choice is currently fully automatic (matched by name), not a user-facing choice as originally specced. Tracked separately as an open gap, not resolved by this amendment.

### Status

Retested — confirmed 2026-07-19, see `uj4-validation-record.md`.

---

### Amendment ID

AMEND-2026-07-19-004

### Amendment title

`Compose_SafeSectionName` naming mismatch vs `FB-F01` — duplicate OneNote sections

### Reason for amendment

`Compose_SafeSectionName` (recurring path, built as part of AMEND-2026-07-19-003) produced different section names than `FB-F01` (the one-off path's equivalent, empty-title-guarded logic): missing the `"Mtg - "` prefix, an unwanted `trim()`, a different character-replacement set, and a different truncation length (50 vs 43). This caused duplicate OneNote sections to be created for what should have been the same section.

### Affected area

- Flow B
- OneNote

### Affected user journey

- UJ3
- UJ4

### Affected baseline files

- None beyond the flow definition itself — this is an internal consistency defect between two expressions that must stay in sync.

### Required design correction

Section-naming logic must be identical across the one-off and recurring paths, since both may resolve to the same physical section over a meeting's lifetime (a one-off meeting can never become recurring here, but the naming convention itself must match for consistency and to avoid future drift).

### Required build correction

Rewrote `Compose_SafeSectionName` to be an exact mirror of `FB-F01`'s logic, reading from `outputs('Compose_SectionDisplayName')`.

### Required test update

After clearing duplicate sections and mapping rows created during debugging, confirm exactly one section is created/reused per series across repeated captures.

### SharePoint mirror update required

Yes — flag that `FB-F01` and `Compose_SafeSectionName` are a linked pair; changing one without the other will silently reintroduce duplicate sections.

### Status

Retested — confirmed 2026-07-19.

---

### Amendment ID

AMEND-2026-07-19-005

### Amendment title

`Set_varOutStatus` regressed to empty string — false "something went wrong" errors

### Reason for amendment

`Set_varOutStatus` was found set to `""` instead of `"OK"`, causing the agent to report "something went wrong" in chat despite Flow B having completed successfully. This is a recurrence of an identical bug pattern first found and fixed on 2026-06-26 (documented in `uj1-validation-record.md`), indicating `OutStatus` is a fragile single hardcoded value with no structural protection against being blanked out again.

### Affected area

- Flow B

### Affected user journey

- UJ3
- UJ4

### Affected baseline files

- `01-shared-contract/shared-journey-contract-vfinal.md`

### Required design correction

Longer-term, `OutStatus` should be set explicitly at each real branch outcome rather than relying on one shared hardcoded value — see the separate, not-yet-actioned item on `OutStatus` differentiation (six-state contract) raised in `2026-07-20-gap-analysis-original-brief-vs-current-build.md`.

### Required build correction

Reset `Set_varOutStatus` to `"OK"`.

### Required test update

Add an explicit assertion to the UJ3/UJ4 test scripts checking `OutStatus = "OK"` on a successful run, not just the absence of a chat error message.

### SharePoint mirror update required

No

### Status

Retested — confirmed 2026-07-20. Root architectural fragility remains open, see AMEND item pending for `OutStatus` differentiation.

---

### Amendment ID

AMEND-2026-07-20-001

### Amendment title

Date-jump day-navigation feature (`C6C_Check_Date`)

### Reason for amendment

The candidate-list prompt has always advertised a third navigation option — "...or a date (e.g. 28 Jun) to jump" — that was never implemented. Any non-P/N input was passed straight through into Flow A's number-selection field and would crash. Discovered while attempting to test UJ5.

### Affected area

- Meeting Capture topic

### Affected user journey

- UJ5 (this feature directly enabled UJ5's first validation)
- Regression / enhancement (extends beyond original UJ1–UJ5 scope)

### Affected baseline files

- `02-user-journeys/user-journey-5-no-match-recovery.md` (original spec described a different recovery menu — "search again with different wording" / "Stop" — not day-jump navigation; this file is now out of date and should be updated once the combined final design, agreed 2026-07-20, is built)

### Required design correction

Confirmed as an accepted design improvement over the original UJ5 recovery menu. Agreed next step (not yet built): add a same-day reword/retry option and an explicit "Stop" exit alongside the existing P/N/date navigation, rather than replacing the original idea outright.

### Required build correction

Added `C6C_Check_Date` as a sibling condition to `C6_Check_Input`/`C6B_Check_N`, using `IsError(Value(Topic.TopicSelectedNumber)) && !IsError(DateValue(Topic.TopicSelectedNumber))` to distinguish dates from numeric selections without regressing existing P/N/number-selection behaviour.

### Required test update

Live tests for partial date ("28 Jun"), full date with year ("10 Feb 2030"), and no-regression confirmation on plain numeric selection — all completed.

### SharePoint mirror update required

Yes

### Status

Retested — confirmed 2026-07-20, see `2026-07-20-date-jump-feature-and-uj-validation.md`. Follow-up build item (reword/retry + Stop) remains open, not yet started.
