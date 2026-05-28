[copilot-instructions.md](https://github.com/user-attachments/files/28370278/copilot-instructions.md)
# GitHub Copilot Instructions — Teams-OneNote-Meeting-Capture

## Project context

This repository documents and guides the build of the Teams-OneNote-Meeting-Capture project.

The project is a Microsoft Copilot Studio solution with Agent Flows / Power Automate-backed flows that resolve Outlook meetings and write structured meeting notes into OneNote.

## Critical terminology

Use these terms consistently:

- **Agent Flow** = a physical flow built in Copilot Studio → Flows / Agent flows.
- **User journey** or **scenario path** = the route the user follows through the Copilot Studio topic.
- **Topic branch** = the conditional path inside the Copilot Studio topic.
- **Flow A** = physical Agent Flow for Outlook meeting lookup only.
- **Flow B** = physical Agent Flow for OneNote capture and recurring mapping.

Do not call user journeys “flows” unless explicitly referring to a physical Agent Flow.

## Build discipline

When suggesting build steps:

1. State whether the step is built in **AGENT**, **FLOW**, or **TOPIC**.
2. Label every value as **Expression**, **Dynamic content**, or **Plain text**.
3. Use exact action names from the relevant artefact.
4. Avoid generic action names such as `Compose`, `Condition`, or `Apply_to_each`.
5. Do not suggest renaming actions after expressions reference them.
6. Prefer small validated baselines over large untested builds.
7. Do not add enrichment to a baseline unless the artefact explicitly says it is in scope.

## Flow A rules

Flow A is `PA - Resolve Meeting Selection - v1 Clean Build`.

Flow A must:

- Use Office 365 Outlook only.
- Use Get calendar view of events.
- Return no match, single match, or multiple matches.
- Return all outputs as strings.
- Include `FA03A_DEBUG_RawConnectorOutput` during first-run schema inspection.

Flow A must not:

- Use OneNote.
- Use SharePoint.
- Call Flow B.
- Include attendee extraction in v1.
- Perform recurring meeting setup.
- Create or update OneNote pages.

## Flow B rules

Flow B owns OneNote and SharePoint mapping.

Flow B must include the OneNote `/pages` safeguard:

- Before Create page in a section, normalise the target section reference into `TargetSectionPagesUrl`.
- `TargetSectionPagesUrl` must end with `/pages`.
- Create page uses the section pages URL.
- Update/append existing page uses the existing page self URL or page identifier required by the update action.

## Testing rules

For every test case, include:

- Test name.
- Purpose.
- Inputs.
- Expected outputs.
- Whether each input is Plain text, Dynamic content, or Expression.

## Response style

Be concise, explicit, and build-ready. Avoid speculative rebuilds. If something is not proven, say it is not proven and define the validation step.
