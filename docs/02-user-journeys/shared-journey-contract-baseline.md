# Shared Journey Contract Baseline

## Teams-OneNote-Meeting-Capture

## 1. Purpose

This artefact defines the shared contract layer for **Agent 1 — Meeting Capture**.

It sits above the individual user journeys so that each journey has awareness of the others before detailed build design begins.

The goal is to avoid designing one user journey in isolation and later discovering that another journey requires a different Topic, Flow A, or Flow B contract.

## 2. Core terminology

### Physical Agent Flow

A physical Agent Flow is built in:

```text
Copilot Studio → Flows / Agent flows
```

Examples:

```text
Agent Flow A — Resolve Meeting Selection
Agent Flow B — Resolve OneNote Section and Capture Notes
```

### User journey

A user journey is the route the user follows through the Copilot Studio topic.

Examples:

```text
User Journey 1 — One-off meeting, single match
User Journey 2 — Multiple meetings, user selects one
User Journey 3 — Recurring meeting, existing mapping found
User Journey 4 — First-time recurring meeting setup
User Journey 5 — No match / recovery
```

### Topic branch

A topic branch is the decision path inside the Copilot Studio topic.

Examples:

```text
If Status = NO_MATCH
If Status = SINGLE_MATCH
If Status = MULTIPLE_MATCHES
If IsRecurring = "true"
If OutRequiresSetup = "true"
```

## 3. Shared architecture

```text
User
  ↓
Copilot Studio Topic
  ↓
Agent Flow A — Resolve Meeting Selection
  ↓
Topic confirmation / selection / branching
  ↓
Agent Flow B — Resolve OneNote Section and Capture Notes
  ↓
OneNote page created or updated
  ↓
User receives OneNote page link
```

## 4. Physical Agent Flow responsibilities

### Agent Flow A — Resolve Meeting Selection

Physical flow name:

```text
PA - Resolve Meeting Selection - v1 Clean Build
```

Flow A owns:

```text
- Outlook calendar lookup
- Meeting candidate discovery
- No match / single match / multiple match status
- Meeting metadata extraction
- Recurrence detection, best effort
- Selected-number resolution for UJ2 in Flow A v3.2
- String outputs back to the topic
```

Flow A does not own:

```text
- OneNote
- SharePoint
- Page creation
- Page update
- Recurring setup questions
- Attendee extraction in v1
- User-facing conversation
```

### Agent Flow B — Resolve OneNote Section and Capture Notes

Flow B owns:

```text
- OneNote section resolution
- OneNote page creation
- OneNote page append/update
- SharePoint recurring mapping lookup
- SharePoint recurring mapping persistence
- Existing recurring page detection
- First-time recurring setup execution
- /pages normalisation before Create page in a section
```

Flow B does not own:

```text
- Outlook meeting lookup
- Asking the user which meeting they meant
- Multiple meeting selection conversation
- Initial PageHtml content composition in v1
```

## 5. Shared user journey map

| Journey | Name | Flow A required? | Flow B required? | Key dependency |
|---:|---|---|---|---|
| UJ1 | One-off meeting, single match | Yes | Yes | Flow A returns `SINGLE_MATCH` |
| UJ2 | Multiple meetings, user selects one | Yes | Yes after selection and confirmation | Selected number must resolve to one meeting |
| UJ3 | Recurring meeting, existing mapping found | Yes | Yes | Flow B finds existing mapping |
| UJ4 | First-time recurring meeting setup | Yes | Yes, possibly twice | Flow B returns setup required |
| UJ5 | No match / recovery | Yes | No | Topic must not call Flow B |

## 6. Topic → Flow A contract

### Flow A v3.1 inputs

| Input | Type | Value type | Source | Notes |
|---|---|---|---|---|
| `InUserSearchText` | Text | Dynamic content | User request text | Original user request |
| `InDateContext` | Text | Plain text | Topic default | v1 uses `today` |
| `InMaxCandidates` | Text | Plain text | Topic default | v1 uses `5` |

### Flow A v3.2 addition for UJ2

| Input | Type | Value type | Source | Notes |
|---|---|---|---|---|
| `InSelectedNumber` | Text | Dynamic content or Plain text | User selected number | Blank on first Flow A call. Populated on second Flow A call for UJ2 selected-number resolution. |

### Versioning rule

```text
Flow A v3.1 = lookup and candidate display.
Flow A v3.2 = lookup, candidate display, and selected-number resolution.
```

## 7. Flow A → Topic contract

All Flow A outputs must be **Text/String**.

| Output | Type | Used by journeys | Notes |
|---|---|---|---|
| `Status` | Text | All | `NO_MATCH`, `SINGLE_MATCH`, `MULTIPLE_MATCHES`, `ERROR` |
| `MatchCount` | Text | UJ2, UJ5 | Numeric count returned as string |
| `CandidateList` | Text | UJ2 | User-readable numbered list only |
| `MeetingTitle` | Text | UJ1, UJ2, UJ3, UJ4 | Populated only once one meeting is resolved |
| `CalendarEventId` | Text | UJ1, UJ2, UJ3, UJ4 | Required before Flow B |
| `IsRecurring` | Text | UJ1, UJ3, UJ4 | `true` or `false` as string |
| `SeriesMasterId` | Text | UJ3, UJ4 | Best effort; may be blank |
| `Start` | Text | UJ1, UJ2, UJ3, UJ4 | `start.dateTime` only |
| `End` | Text | UJ1, UJ2, UJ3, UJ4 | `end.dateTime` only |
| `Location` | Text | UJ1, UJ2, UJ3, UJ4 | `location.displayName` only |
| `AttendeesSummary` | Text | All | Always empty string in Flow A v1 |
| `OutErrorMessage` | Text | All | Empty unless error path added |

## 8. Shared topic variables

| Topic variable | Source | Purpose |
|---|---|---|
| `Topic.FlowAStatus` | `FlowA.Status` | Main branch decision |
| `Topic.MatchCount` | `FlowA.MatchCount` | Multiple match validation |
| `Topic.CandidateList` | `FlowA.CandidateList` | Display candidate choices |
| `Topic.MeetingTitle` | `FlowA.MeetingTitle` | Selected meeting title |
| `Topic.CalendarEventId` | `FlowA.CalendarEventId` | Selected event ID |
| `Topic.IsRecurring` | `FlowA.IsRecurring` | One-off vs recurring branch |
| `Topic.SeriesMasterId` | `FlowA.SeriesMasterId` | Recurring mapping key, best effort |
| `Topic.Start` | `FlowA.Start` | User confirmation / page content |
| `Topic.End` | `FlowA.End` | User confirmation / page content |
| `Topic.Location` | `FlowA.Location` | Page content / display |
| `Topic.SelectedNumber` | User answer | UJ2 selection |
| `Topic.OutErrorMessage` | `FlowA.OutErrorMessage` | Safe error handling |

## 9. Topic branching contract

### No match branch

Condition:

```text
Topic.FlowAStatus = "NO_MATCH"
```

Topic behaviour:

```text
- Do not call Flow B
- Show recovery options
- Allow user to retry or stop
```

Applies to:

```text
UJ5
```

### Single match branch

Condition:

```text
Topic.FlowAStatus = "SINGLE_MATCH"
```

Topic behaviour:

```text
- Store meeting details
- Check Topic.IsRecurring
- Continue to one-off or recurring path
```

Applies to:

```text
UJ1
UJ3
UJ4
UJ2 after selected meeting resolution
```

### Multiple matches branch

Condition:

```text
Topic.FlowAStatus = "MULTIPLE_MATCHES"
```

Topic behaviour:

```text
- Show CandidateList
- Ask user to select a number
- Validate the selection
- Resolve selected meeting using second Flow A call with InSelectedNumber
- Confirm the resolved meeting with the user
- Do not call Flow B until one meeting is resolved and confirmed
```

Applies to:

```text
UJ2
```

#### Required topic validation before second Flow A call

Before calling Flow A a second time, the topic must validate:

```text
Topic.SelectedNumber is not blank
Topic.SelectedNumber is numeric
Topic.SelectedNumber >= 1
Topic.SelectedNumber <= Topic.MatchCount
```

If validation fails:

```text
Do not call Flow A again.
Do not call Flow B.
Ask the user to choose a number from the displayed list.
```

### Error branch

Condition:

```text
Topic.FlowAStatus = "ERROR"
```

Topic behaviour:

```text
- Show safe error message
- Do not expose connector JSON
- Do not call Flow B
```

Applies to:

```text
All journeys
```

## 10. Topic → Flow B contract

Flow B should only be called after the topic has resolved and confirmed one meeting.

| Flow B input | Type | Value type | Source | Notes |
|---|---|---|---|---|
| `MeetingTitle` | Text | Dynamic content | `Topic.MeetingTitle` | Must be populated |
| `MeetingId` | Text | Dynamic content | `Topic.CalendarEventId` | Must be populated |
| `IsRecurring` | Text | Dynamic content | `Topic.IsRecurring` | `true` or `false` as string |
| `SeriesMasterId` | Text | Dynamic content | `Topic.SeriesMasterId` | May be empty |
| `PageHtml` | Text | Expression / Dynamic content | Topic-generated page HTML | Topic owns initial PageHtml generation in v1 |
| `SectionChoice` | Text | Plain text or Dynamic content | Topic setup answer | Empty for normal path |
| `SectionName` | Text | Plain text or Dynamic content | Topic setup answer | Empty unless setup |
| `UpdateType` | Text | Plain text | Topic branch | e.g. `NOTES`, `SETUP_RECURRING` |

### Flow B call rule

Do **not** call Flow B unless:

```text
Topic.FlowAStatus = "SINGLE_MATCH"
Topic.MeetingTitle is not empty
Topic.CalendarEventId is not empty
The selected/resolved meeting has been confirmed with the user when required
```

## 11. PageHtml ownership

The Copilot Studio topic owns initial `PageHtml` generation for Agent 1 v1.

The topic builds a clean HTML block from:

```text
- selected meeting title
- selected meeting start and end
- location if available
- user-provided notes or captured note content
- reserved sections for later summary/transcript enrichment if required
```

Flow B receives `PageHtml` as an input and persists it to OneNote.

Flow B owns OneNote page create/update mechanics, not the user-facing content decision in v1.

This may be revisited later if HTML generation becomes complex enough to justify a separate helper flow.

## 12. Flow B → Topic contract

All Flow B outputs should be **Text/String**.

| Flow B output | Type | Used by journeys | Notes |
|---|---|---|---|
| `OutStatus` | Text | UJ1, UJ3, UJ4 | `SUCCESS`, `RECURRING_SETUP_REQUIRED`, `ERROR` |
| `OutRequiresSetup` | Text | UJ3, UJ4 | `true` or `false` as string |
| `OutPageAction` | Text | UJ1, UJ3, UJ4 | `PAGE_CREATED`, `PAGE_UPDATED_APPEND`, etc. |
| `OutPageLink` | Text | UJ1, UJ3, UJ4 | User-facing OneNote link |
| `OutAgentResponseSummary` | Text | UJ1, UJ3, UJ4 | Clean response summary |
| `OutErrorMessage` | Text | All Flow B journeys | Empty unless error |

## 13. Shared Flow B rules

Flow B must enforce:

```text
- OneNote /pages normalisation before Create page in a section
- Existing page append path must not create duplicate pages
- Existing recurring mappings must be reused
- First-time recurring setup must persist reusable mapping
- Blank SeriesMasterId must be handled safely
```

## 14. Cross-journey dependency register

| Dependency | Affects | Risk | Design response |
|---|---|---|---|
| `CandidateList` is display-only | UJ2 | User selection cannot resolve to meeting | Adopt Option A: second Flow A call with `InSelectedNumber` |
| Candidate array is rebuilt on second Flow A call | UJ2 | Meeting could be created, cancelled, deleted, or changed between calls, causing selected number to resolve differently | Topic shows resolved meeting title/time for confirmation before Flow B |
| `SeriesMasterId` may be blank | UJ3, UJ4 | Recurring mapping key may be unavailable | Use best-effort fallback and Flow B safe handling |
| OneNote section URL may lack `/pages` | UJ1, UJ3, UJ4 | Create page may fail | Flow B normalises `TargetSectionPagesUrl` |
| Flow A output types | All | Topic type mismatch | All outputs are strings |
| Attendee schema unproven | All | Extra loop risk | Attendees deferred to v2 |
| No match path | UJ5 | Accidental Flow B call | Topic blocks Flow B unless `SINGLE_MATCH` and required fields are populated |
| PageHtml ownership unclear | UJ1, UJ3, UJ4 | Topic/Flow B responsibility blur | Topic owns initial `PageHtml` generation in v1 |

## 15. Current journey readiness

| Journey | Status | Notes |
|---|---|---|
| UJ1 — One-off meeting, single match | Conceptually complete | Depends on Flow A v3.1 and Flow B baseline |
| UJ2 — Multiple meetings, user selects one | Ready for deep design after this contract update | Requires Flow A v3.2 with `InSelectedNumber` |
| UJ3 — Recurring existing mapping | Not yet fully designed | Depends on Flow B mapping logic |
| UJ4 — First-time recurring setup | Not yet fully designed | Most complex path |
| UJ5 — No match / recovery | Partially covered by Flow A v3.1 | Needs topic recovery design |

## 16. Accepted UJ2 design direction

The accepted design direction for User Journey 2 is:

```text
Option A — second Flow A call with InSelectedNumber
```

Rejected for v1:

```text
Option B — hidden CandidatePayloadJson, due to topic-side JSON parsing risk
Option C — fixed Candidate1/Candidate2 fields, due to output contract clutter and maintenance debt
```

## 17. Next design step

Proceed to:

```text
User Journey 2 — Multiple meetings, user selects one — Deep Design v1
```

Deep design must include:

```text
1. First Flow A call
2. CandidateList display
3. User selection capture
4. Topic validation
5. Second Flow A call with InSelectedNumber
6. Resolved meeting confirmation
7. Branch to one-off or recurring path
8. Rule that Flow B is not called until confirmation
9. Test cases
10. Claude review prompt for UJ2 deep design
```
