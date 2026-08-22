# Plan for next session — after 22 August 2026 evening

**Written:** end of 22 August 2026 evening session.
**Read first:** `session-2026-08-22-evening-uj345.md` and `CURRENT-STATE.md`.

---

## Priority 1: Investigate recurring meeting capture errors

David is observing occasional errors on 2nd/3rd captures from the same recurring series. Root cause is unknown. **Do not build anything until the root cause is confirmed from evidence.**

**Step 1:** Trigger a capture on a recurring series that has previously failed. Let it complete (or fail).

**Step 2:** Pull the Activity trace from Flow B for that run. Check in this order:
- Did `Get_items` return the mapping row? (check `body.value` — should be non-empty)
- Did `Filter_Existing_Mapping` return a match? (check output count)
- Did `Compose_PageDecision` output `PAGE_EXISTS` or `PAGE_NOT_FOUND`?
- If `PAGE_EXISTS`: did `Filter_Pages_By_Title` return a match? (check output count)
- Did `Compose_RealExistingPageId` return a real ID or empty string?
- Did `Update_page_content_Existing_Branch` succeed or fail?
- What did `Set_varOutStatus` evaluate to?

**Likely root causes to look for:**
- `Get_items` returning empty (transient caching issue — seen before on 21 Aug)
- `Filter_Pages_By_Title` returning empty (page title format mismatch — e.g. old undated pages)
- Corruption having struck `Set_varOutStatus` or another key action between sessions

**Model/effort:** Sonnet 4.6, Standard for diagnosis. Switch to High only if root cause is genuinely ambiguous after reviewing the trace.

---

## Priority 2: UJ3b — Automatic stale-row cleanup

When `STALE_MAPPING` is detected, automatically delete or update the stale mapping row so the next capture takes the `CREATE_REQUIRED` path without user intervention.

**Design (already reasoned through):**
- Add a Condition inside `Apply_to_each_Existing_Section` checking whether `Compose_RealExistingPageId` returned empty
- True branch: SharePoint `Update item` to clear the stale row's `PageSelfUrl`
- False branch: existing update logic (unchanged)
- `Update_page_content_Existing_Branch` runAfter updated to only run when page ID is non-empty

**Risk:** structural edit 3 levels deep in the canvas. Take a fresh Peek Code snapshot before starting. Run Flow Checker immediately after each structural change.

**Model/effort:** Sonnet 4.6, Standard.

---

## Priority 3: UJ4c — SectionRetryCount retry loop

Do Until retry loop around `Create_Section_Recurring` for transient section creation failures.

**Design:** requires 2 new InitializeVariable actions at top of flow + Do Until wrapping the section creation. Higher corruption risk than UJ3b. Defer until recurring errors investigation and UJ3b are complete.

**Model/effort:** Sonnet 4.6, Standard.

---

## Don't forget

- Run a fresh **Flow checker** at the start of the session before touching anything — corruption may have struck between sessions.
- Pull fresh Peek Code on `Set_varOutStatus` specifically — it has been the most frequent corruption target.
- Microsoft discussion brief is ready in `microsoft-discussion-brief-corruption-bug.md` for the in-person meeting next week.

---
*Written 22 August 2026, end of evening session.*
