# Bug 8 — outStatus/outCreatedPageLink empty on brand-new-section creation path

## Status: DIAGNOSED (structural cause identified), not yet fixed. Logged for next session.

## Found during

Live testing after today's SharePoint `PageWebUrl` fix (see `handover-2026-08-08-INCIDENT-pagewebrul-badgateway.md`) — the underlying flow run **succeeded** (green, no error, `Respond to the agent` returned `statusCode: 200`), but Teams displayed: *"I'm sorry, something went wrong saving your meeting notes. Please try again."*

The OneNote page was genuinely created correctly — this is purely a reporting/output problem, not a data-loss or capture-failure problem.

## Root cause of the Teams-side false failure

`Respond to the agent`'s output body showed:
```json
"outstatus": "",
"outcreatedpagelink": "",
"outcreatedpageselfurl": "",
```
All three blank. The Copilot Studio topic evidently branches on `outstatus` (or similar) to decide which message to show the user — with it empty, the topic falls into its "something went wrong" fallback, even though nothing actually went wrong.

## Which path this affects

Traced via Activity, run 8/8 12:07 PM. The run took this route:
`Condition Mapping Exists` → **False** (no existing mapping — first time this meeting/section has ever been captured) → `Condition Section Exists Recurring` → **False** (brand new section) → `Condition Should Write Mapping` → **True** → `Condition Should Create Page` → **True** → `Create OneNote Page` → ... → `Respond to the agent`.

Output confirms: `outonenoteresolverresult: "CreatedSection"`, `outpageroute: "False"`.

**This is a third, distinct branch** from the two we worked on today (recurring-existing-mapping, one-off-existing-mapping) — it's the "very first capture of a brand-new section" path. Bug 7 and today's `PageWebUrl` fix never touched this branch at all.

## Confirmed structurally via canvas trace

- `Set varOutStatus` **does exist** in the flow, positioned right at the end (`Compose AgentResponseSummary` → `Compose SP Item Count` → `Set varOutStatus` → `Respond to the agent`) — so it should run on every path, including this one.
- **This means the action isn't missing** — the expression *inside* `Set varOutStatus` (and likely a sibling issue in whatever sets `varOutputPageLink`/`varOutputPageSelfUrl`) doesn't correctly cover the `CreatedSection` branch's conditions, so it evaluates to empty/falls through on this specific path.
- **For comparison, a working example exists on a related path**: the one-off "create new page" branch (`Create Page OneOff` → `Set varOutputPageLink Created OneOff`) *does* correctly set its output link — confirmed via canvas trace in the same session. So this isn't a universal problem across all "create" paths, just this specific brand-new-section-creation one.

## Not yet done

- Have not opened `Set varOutStatus`'s actual expression via Peek Code to see its exact condition logic and confirm precisely why the `CreatedSection` case falls through.
- Have not checked whether `varOutputPageLink` / `varOutputPageSelfUrl` have the same gap on this specific branch, or whether only `varOutStatus` is affected — worth checking during the fix rather than assuming.
- Have not checked whether this is a long-standing pre-existing gap (likely, given it doesn't relate to anything touched in recent sessions) or something recently introduced — no evidence either way yet.

## Impact

**User-facing false failure message on a working capture.** This is a real usability bug, not cosmetic — it happens on the very first time any new section/meeting is captured (i.e. every genuinely new recurring meeting series' first occurrence), which is a common, ordinary usage pattern, not an edge case.

## Fix direction (not yet built)

Once diagnosed further: locate `Set varOutStatus`'s expression, add explicit handling for the `CreatedSection` case (and confirm/add for `varOutputPageLink`/`varOutputPageSelfUrl` if the same gap exists there), test via a fresh brand-new meeting capture live before publishing.

## Status

**Diagnosed to the specific branch and confirmed the action exists but its logic doesn't cover this case. Not yet fixed — deliberately deferred to a fresh session given the length of today's session (power loss, corruption incident, BadGateway incident, and this finding all in one sitting).**
