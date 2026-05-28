# Claude Review Instructions

Use this file when asking Claude or another reviewer to assess this project.

The review style should be critical, not supportive by default.

For every proposed Agent Flow, review:

1. Connector choice.
2. Action order.
3. Field mappings.
4. Power Automate expression validity.
5. Null safety.
6. Object-to-string risks.
7. Duplicate or unstable action names.
8. Whether outputs are strings.
9. Whether the flow is too complex for a first baseline.
10. Recommended simplifications before build.
11. Design confidence, first-build confidence, and production confidence after validation.

Do not assume connector schemas are fully reliable until a debug output confirms the real run payload.
