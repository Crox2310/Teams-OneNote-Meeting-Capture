# Agent 1 User Journeys / Scenario Paths

## Important distinction

These are user journeys, not physical Agent Flows.

A physical Agent Flow is built in Copilot Studio → Flows / Agent flows.
A user journey is the route the user follows through the agent experience.

## User journeys

| # | User journey | Description |
|---:|---|---|
| 1 | One-off meeting, single match | Agent finds one meeting and proceeds to capture notes |
| 2 | Multiple meetings, user selection | Agent shows a numbered list and user selects one |
| 3 | Recurring meeting, existing mapping found | Agent uses stored OneNote location silently |
| 4 | First-time recurring meeting setup | Agent asks where future notes for the series should be stored |
| 5 | No match / recovery | Agent cannot find a meeting and offers recovery options |

## Journey-to-flow mapping

| User journey | Calls Flow A? | Calls Flow B? | Requires user selection? | Requires recurring setup? |
|---|---|---|---|---|
| One-off meeting, single match | Yes | Yes | No | No |
| Multiple meetings, user selection | Yes | Yes after selection | Yes | Maybe |
| Recurring meeting, existing mapping found | Yes | Yes | No | No |
| First-time recurring meeting setup | Yes | Yes, possibly twice | Maybe | Yes |
| No match / recovery | Yes | No | Maybe later | No |

## Topic branching logic

```text
Call Flow A
↓
If Status = NO_MATCH
    → Journey 5: No match / recovery

If Status = MULTIPLE_MATCHES
    → Journey 2: User selects meeting
    → Resolve selected meeting
    → Continue to one-off or recurring path

If Status = SINGLE_MATCH
    → Check IsRecurring

        If IsRecurring = "false"
            → Journey 1: One-off single match

        If IsRecurring = "true"
            → Call Flow B
                If OutRequiresSetup = "false"
                    → Journey 3: Recurring existing mapping
                If OutRequiresSetup = "true"
                    → Journey 4: First-time recurring setup
```
