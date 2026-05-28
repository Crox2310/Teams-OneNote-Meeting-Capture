# Claude Flow Review Template

Use this prompt to ask Claude to stress-test an Agent Flow.

```text
I want you to simulate this Microsoft Copilot Studio Agent Flow / Power Automate run in memory.

Please act as a critical technical reviewer, not a supportive reviewer.

Assess:
1. Whether the connector choice is right.
2. Whether the action order is correct.
3. Whether the field mappings are safe.
4. Whether expressions are valid Power Automate syntax.
5. Whether any object could be passed into a string field.
6. Whether nulls or missing fields could terminate the flow.
7. Whether all outputs are clean strings.
8. Whether the flow is too complex for a first baseline.
9. What should be removed from v1.
10. First-build confidence percentage.
11. Production confidence after validation.
12. Required corrections before build.

Please provide:
- Step-by-step simulation.
- Likely failure points.
- Corrected expressions where needed.
- Final confidence scores.
- Recommendation to build as-is or simplify.
```
