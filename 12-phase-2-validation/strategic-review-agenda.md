# Strategic review agenda — post-completion

**Written:** 22 August 2026, end of day.
**Purpose:** once the remaining bugs (BUG-01, general recurring errors) and feature requests are closed out, David wants to step back and conduct a structured review of the project as a whole. This document captures the agenda for that conversation.

---

## 1. Original brief vs. what was built

**Question:** did we meet the original brief? Where did we deviate, and were those deviations intentional improvements or compromises?

**Context:** the brief evolved significantly during development — the original scope was simpler (basic meeting capture to OneNote) and expanded to include recurring meeting support, per-occurrence pages, OccurrenceDate tracking, the full OutStatus framework, and UJ3-5 coverage. Worth mapping what was originally asked for against what was delivered, and whether anything from the original brief was quietly dropped.

**Preparation:** pull the original design brief / early handover docs to reconstruct the original scope before the review session.

---

## 2. Architecture review: did we build it the right way?

**Question:** in hindsight, is the single-agent / two-flow architecture the right design? What would we do differently?

**Specific angles to explore:**
- **Smaller, orchestrated agents** — should this have been broken into multiple specialised agents (e.g. a meeting-finder agent, a OneNote-writer agent, a mapping-manager agent) orchestrated by a parent? Would that have reduced complexity, improved maintainability, and reduced the blast radius of the corruption pattern?
- **Flow A vs Flow B split** — is the current split (Flow A = meeting selection, Flow B = OneNote write) the right boundary? Should the SharePoint mapping logic live in its own flow?
- **SharePoint as the mapping store** — was SharePoint the right choice for the mapping table, given the BadGateway issues and the read/write complexity? Would Dataverse, a simple JSON blob in OneDrive, or even a Topic variable have been simpler?
- **The corruption surface** — would a different architecture (e.g. solution-exported flows edited via VS Code rather than the Designer) have avoided the corruption pattern entirely?

---

## 3. User experience review

**Question:** is the current user experience (screens, messages, flow of conversation) as good as it could be?

**Specific angles to explore:**
- The candidate list format — is it clear and scannable? Could it be improved with meeting time, location, or organiser shown?
- The navigation model (P/N/date/number/C) — is it intuitive? Would a different interaction model work better?
- The success message and link — is it useful? (FR-03 link shortening is related here)
- Error messages — are they actionable and clear? (STALE_MAPPING, SETUP_SECTION_AMBIGUOUS etc.)
- The retry flow (R to retry) — does it feel natural or clunky?
- Could any of this be replaced with an Adaptive Card for a richer UI experience in Teams?

---

## 4. Performance review

**Question:** how fast is the agent, and where are the bottlenecks? What could be done to speed it up?

**Known slow points to investigate:**
- `FA08 Get calendar view of events` — the Microsoft Graph calendar call. Can the time window be narrowed? Is there a caching opportunity?
- `Get_items` (SharePoint) — currently retrieves all rows with no filter. Adding an OData `$filter` on `SeriesMasterId` would reduce payload size and improve speed. (Flow checker already flags this as a warning.)
- The 5-second `Delay_Post_Page_Creation` — necessary to avoid the OneNote indexing race condition, but adds 5 seconds to every new-page capture. Could a poll-until approach replace it with a shorter average wait?
- `Set_PageTitle_Recurring` — the post-creation title update is a second API call. Could the title be set at creation time instead, avoiding this call entirely?
- Flow B's overall action count — the flow has grown significantly. Are there redundant actions or paths that could be pruned?

---

## 5. SharePoint mapping table — data retention

**Question:** do we actually need to keep mapping rows indefinitely? Could rows be automatically deleted after a certain period or once they're no longer useful?

**Context:** the `RecurringMeetingSectionMap` list grows with every new recurring series captured. Rows contain `OccurrenceDate`, `SeriesMasterId`, `PageSelfUrl`, `PageWebUrl`, and related fields. Their purpose is to route subsequent captures of the same occurrence to the correct existing page.

**Angles to explore:**
- **Retention period** — after how long is a mapping row no longer useful? Once a meeting occurrence has passed (e.g. more than 30/60/90 days ago), it's unlikely to be recaptured. Rows older than a threshold could be deleted automatically.
- **Automatic cleanup** — could a scheduled flow (e.g. weekly) delete rows where `OccurrenceDate` is older than N days?
- **Whether the mapping table is needed at all** — the mapping table exists because Flow B needs to know whether a page already exists for a given occurrence. Could this instead be determined by querying OneNote directly (search for a page with the expected title) rather than maintaining a separate lookup table? This would eliminate the mapping table entirely but add a OneNote search call to every capture.
- **Privacy/data minimisation** — is there a data governance reason to limit retention? The rows contain meeting titles and OneNote URLs which are personal/work data.

---

## Suggested format for the review session

This is a reflective/strategic conversation, not a build session. Suggested approach:
- No flow editing during this session
- Work through each section above in order
- For each section: what's good, what would we do differently, what (if anything) should be changed
- Produce a short set of recommendations at the end
- Only then decide which recommendations to actually implement

**Model/effort for this session:** Sonnet 4.6, High effort — this is design and reasoning work, not mechanical building.

---
*Written 22 August 2026. Schedule after BUG-01 and remaining feature requests are closed.*
