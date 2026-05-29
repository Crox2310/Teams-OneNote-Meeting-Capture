# UJ2 Pre-Design Readiness Checklist

## Purpose

Use this checklist before deep-designing **User Journey 2 — Multiple meetings, user selects one**.

## Shared contract updates

- [ ] `InSelectedNumber` added to the Flow A v3.2 input contract.
- [ ] Multiple-matches branch states that the topic validates selected number before second Flow A call.
- [ ] Validation rule includes non-blank, numeric, greater than/equal to 1, less than/equal to MatchCount.
- [ ] Cross-journey dependency register includes candidate array stability risk.
- [ ] Topic confirmation step added before Flow B call.
- [ ] PageHtml ownership note added to shared contract.

## UJ2 accepted design direction

- [ ] Option A is accepted.
- [ ] Option B is rejected for v1.
- [ ] Option C is rejected for v1.

## Flow A versioning

- [ ] Flow A v3.1 remains valid for UJ1 and UJ5 basic paths.
- [ ] Flow A v3.2 is required for full UJ2 selection resolution.
- [ ] Flow A v3.2 does not add OneNote, SharePoint, attendees, or recurring setup.

## Topic safeguards

- [ ] Topic does not call Flow B after the first `MULTIPLE_MATCHES` response.
- [ ] Topic asks user to select a number.
- [ ] Topic validates selection before second Flow A call.
- [ ] Topic confirms the resolved meeting before calling Flow B.

## Ready to proceed?

Proceed to UJ2 deep design only when all items above are checked.
