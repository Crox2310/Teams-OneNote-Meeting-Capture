
# Design idea — OneNote Section-Group lane routing via Outlook category

**Date:** 28 August 2026
**Status:** Idea captured, not started. Nothing built. This document exists to record the design and the validation steps needed *before* any flow changes are attempted, per standing protocol (Peek Code / live evidence before proposing fixes — never build against an assumption).

---

## 1. Problem

OneNote sections created by this agent currently sit flat at the root of the "Meeting Notes" notebook. As the number of captured meetings grows, this becomes hard to navigate — confirmed directly from a screenshot of the live notebook (28 Aug 2026), showing 15+ ungrouped sections with no organisation beyond creation order.

This is the same retrieval problem the personal operating model (`Crox2310/operating-model`) already solved for email and files: a small, fixed set of lanes beats an unbounded flat list. That repo's six-lane taxonomy (Priority, Team, Cross Domain, Supply Chain Tech, Logistics Tech, Programme Enablement) is already confirmed live across Outlook, Calendar, OneNote, and Teams as *colour-coding* (24 Aug 2026) — but Part D of that brief explicitly flags as an open question whether OneNote pages/sections from this agent are actually organised to match, and confirms they are not.

## 2. Proposed design

**OneNote Section Groups**, one per lane, mirroring the six-lane taxonomy exactly:

- Priority
- Team
- Cross Domain
- Supply Chain Tech
- Logistics Tech
- Programme Enablement
- (plus an "Inbox"-equivalent group as a fallback — see Section 4)

Each meeting's OneNote section is created inside the Section Group matching its lane, instead of at notebook root.

**Signal used to determine the lane:** the Outlook calendar event's **colour category**. The operating model already requires every meeting to carry one of the six lane categories (applied to calendar invites since 18 Aug 2026, colours finalised 24 Aug 2026) as part of the Stage 1 flag-and-file habit. This is a better signal than title/attendee keyword-matching (the alternative considered first) because it's an explicit, deliberate tag the user is already maintaining for other reasons — not something the flow has to infer.

**Mapping (proposed, 1:1):**

| Outlook category | Target Section Group |
|---|---|
| Priority | Priority |
| Team | Team |
| Cross Domain | Cross Domain |
| Supply Chain Tech | Supply Chain Tech |
| Logistics Tech | Logistics Tech |
| Programme Enablement | Programme Enablement |
| *(none / uncategorised)* | Inbox (fallback group) |

## 3. Why this looked buildable at first glance

Flow A's shared data contract (`01-shared-contract/outlook-meeting-data-capture-profile-v1.md`) already lists `CategoriesSummary` as an optional field the Outlook connector output can populate — i.e. the category data is, in principle, already reachable from the trigger payload without any new Graph permissions or connector changes.

**This has NOT been confirmed as actually implemented.** A repo-wide code search for `CategoriesSummary` returns exactly one hit — the shared-contract doc itself. It does not appear in any flow-reference Peek Code capture, handover note, or known-good-values reference anywhere else in the repo. This means one of two things is true, and the validation plan below exists specifically to determine which:

- (a) it's implemented in the live flow but has simply never come up in a session note, or
- (b) it's a documented V1 *option* that was never actually switched on / wired through to anywhere downstream.

Treat it as **unconfirmed** until checked directly.

## 4. Open assumptions requiring validation before any build work

1. **Is `CategoriesSummary` actually populated in Flow A today?** Needs direct Peek Code on the Outlook connector trigger/action in Flow A and a live run's raw output (`FA03A_DEBUG_RawConnectorOutput` per the shared contract doc), not just the shared-contract spec.
2. **What does the value actually look like when populated?** Category name string(s), a colour preset id, both, or something else — determines how simple the lane-mapping expression can be.
3. **Does it reach the Topic / Flow B, or does it stop at Flow A?** The capture profile is Flow A's contract; need to confirm this field is actually passed through to wherever the section-creation logic lives (Flow B), not just captured and discarded.
4. **Coverage gap — uncategorised meetings.** Any meeting not yet triaged into a lane category (new invite, or one Stage 1 hasn't reached yet) will have no category at capture time. Needs an explicit fallback (proposed: an "Inbox" Section Group, mirroring the email Inbox lane) rather than silently defaulting into one of the six and filing it wrong.
5. **Multiple categories on one event.** Outlook allows more than one category per item. Needs a defined tie-break rule (e.g. first match against the six lane names, in a fixed priority order) if this occurs in practice.
6. **Section-group-scoped creation mechanics.** `Create_Section_Recurring` / `Create_Section_OneOff` currently create sections at notebook root via the native OneNote connector action. Creating inside a Section Group is a different Graph endpoint (`.../onenote/notebooks/{id}/sectionGroups/{groupId}/sections`) and may require an HTTP action rather than the native connector action — similar in kind to the BadGateway native-connector fix already done elsewhere in this flow, but unconfirmed whether the native connector supports a section-group target at all.
7. **Section-group-scoped lookup mechanics.** Every existing section-matching action (`Get_Sections`, `Filter_OneNote_Section_Recurring`, `Filter_OneNote_Section_OneOff`, `Filter Existing Section By Name`) currently searches the whole notebook. Once sections live inside groups, these need to search within the correct group only — otherwise cross-lane name collisions become possible, and match-count logic (`Compose_SectionMatchCount_*`, feeding `SETUP_SECTION_AMBIGUOUS`) could be affected.

## 5. Validation plan (do this before writing any flow changes)

Following standing protocol — live evidence first, no changes based on assumption:

1. **Peek Code the Outlook connector trigger action in Flow A live**, and pull one real run's raw output. Confirm directly whether `CategoriesSummary` (or an equivalent categories field) is present and populated for a real categorised meeting.
2. **If present:** capture the exact shape of the value (string, array, colour id) in this document, and confirm via a second real run that it's consistent across a recurring meeting and a one-off meeting.
3. **If present:** trace forward through the Topic/Flow B chain to confirm the field is actually reachable at the point section-creation happens, not dropped somewhere in between.
4. **If absent or only partially wired:** determine what's needed to switch it on — likely a Flow A connector action change to explicitly request `categories` from the Graph/connector call, plus a new Compose step to surface it the way `CategoriesSummary` is specified. Scope that as its own small piece of work before the section-group routing work can start.
5. **Separately, and independent of (1)-(4):** test section-group-scoped creation and lookup against `PA - Scratch Diagnostics` — confirm the native OneNote connector actions support a Section Group as a target/scope, or whether an HTTP action against the Graph endpoint is required. Do this against a disposable test notebook/group, not the live "Meeting Notes" notebook.
6. **Only once (1)-(5) are confirmed:** write the actual build design (trigger changes, mapping expression, fallback group, updated lookup scoping) as a follow-up to this document.

## 6. Relationship to other in-flight work

- **Manual reorg (immediate, separate track):** regardless of whether this automation ever gets built, the existing flat section list should be manually sorted into six Section Groups now, for immediate relief. That work does not depend on anything in this document and isn't blocked by it.
- **Phase 2 vision (`phase-2-vision.md`):** this is a distinct piece of work from Phase 2's post-meeting ingestion / pre-day prep features, but shares the same "Phase 1 must be fully stable first" gating logic. Worth sequencing alongside that review rather than as an ad hoc addition.
- **Operating model (`Crox2310/operating-model`, Part D open questions):** this document is the direct follow-up to Part D's open question about whether OneNote pages are organised to match the six-lane structure. Once validated and built, that open question can be closed out in the operating-model repo.

## 7. Next steps

1. Run the validation plan (Section 5) in a session with Designer/Peek Code access.
2. Update this document with findings — do not start build work until Section 4's assumptions are each explicitly confirmed or refuted.
3. If validated, write the build design as a new dated document, following this repo's existing convention (see `2026-07-27-condition-is-genuine-existing-page-defect.md` for the standard of evidence expected before a fix is proposed).

---
*Written 28 August 2026. Nothing in this document has been implemented or tested. Treat every claim about current flow behaviour as unconfirmed until checked live.*
