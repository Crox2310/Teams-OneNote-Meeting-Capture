# Handover — 16 August 2026 (continued) — One-off branch title fix built but NOT YET tested; branch-routing clarified

## START HERE

This continues directly from `handover-2026-08-16-page-title-fix-recurring-confirmed.md`, which confirmed the permanent page-title fix working for the recurring-branch page-creation path. This session built the mirrored fix for the one-off branch (`Create_Page_OneOff`) — same five-action pattern, same discipline — but **discovered mid-test that this code path is much narrower than assumed, and has not yet been genuinely exercised.**

**Do not assume the one-off title fix works. It is built, Flow-Checker-clean, and published, but unconfirmed.**

---

## What was built (mirrors this morning's recurring-branch fix exactly)

Inside `Condition_Is_Genuine_Existing_Page`'s False branch (the "genuinely create a new page" path within the "page already has a mapping row" branch):

1. **`Compose_SafePageTitle_OneOff`** — same sanitisation expression as the recurring version, sourced from `triggerBody()?['text_1']`.
2. **`Create_Page_OneOff`** — unchanged.
3. **`Get_Pages_In_Section_OneOff_PostCreate`** — fresh `GetPagesInSection` read on `variables('varTargetSectionPagesUrl')`.
4. **`Filter_Pages_By_SelfUrl_OneOff`** — filters that live result by matching `self` against `body('Create_Page_OneOff')?['self']` directly (the one-off branch has no separate `Compose_PageSelfUrl_Created`-equivalent action, so this references `Create_Page_OneOff`'s own output body directly rather than an intermediate Compose action).
5. **`Compose_ConfirmedCreatedPageId_OneOff`** — safe extraction with `''` fallback.
6. **`Set_PageTitle_OneOff`** — `UpdatePageContent`, `target: "title"`, `pageId` from step 5, `content` from step 1.

All five actions built via the canvas `+` icon (the reliable method identified earlier today), each confirmed present on canvas before proceeding, each isolated save, Flow Checker clean throughout (0 errors). Published successfully.

## Discovery: this code path is much narrower than assumed

**Test attempted**: fresh one-off capture, `MeetingTitle: OneOff Title Fix Test 16 Aug`, `MeetingId: oneofftitlefix-16aug-1`, `IsRecurring: false` — a brand-new MeetingId with no prior SharePoint row, expecting this to exercise the newly-built one-off title-fix chain.

**What actually happened**: the run trace showed `Compose SafePageTitle` → `Create OneNote Page` → `Get Pages In Section Recurring PostCreate` → `Filter Pages By SelfUrl Recurring` — i.e. **the shared, generic "create a brand new page" path** (`Condition_Should_Create_Page` = True → `Create_OneNote_Page`), not `Create_Page_OneOff` at all, despite `IsRecurring` being explicitly set to `false`.

**Root cause, on inspection**: `Condition_Should_Create_Page` branches on `varFinalPageDecision == 'PAGE_NOT_FOUND'`, which is `true` for **any meeting captured for the first time**, recurring or one-off alike — there is nothing recurring-specific about that condition. `Create_Page_OneOff` only fires in a narrower, rarer scenario, reached via the **opposite** branch: `Condition_Should_Create_Page` = False (a SharePoint mapping row *already exists*, so `varFinalPageDecision == 'PAGE_EXISTS'`) → `Condition_Is_Genuine_Existing_Page` = False (i.e. `varOneNoteResolverResult` is **not** `ExistingMapping` or `ExistingSection` — meaning the mapping row points at a OneNote section that no longer resolves as genuinely existing, e.g. a stale mapping after the section was deleted/renamed). This is conceptually close to the original Bug 5 scenario (stale/orphaned mapping data), not "any ordinary first-time one-off capture."

**Practical implication — genuinely good news for the common case**: every ordinary first-ever one-off meeting capture already routes through `Create_OneNote_Page`, the exact action fixed and confirmed working this morning. **The common, everyday one-off capture path already has working page titles.** `Create_Page_OneOff` and today's new title-fix chain around it only matter for the narrower stale-mapping edge case.

## Current state

- **Page-title gap — ordinary one-off first captures**: already covered by this morning's recurring-branch fix, since both paths converge on the same `Create_OneNote_Page` action for first-time captures. **Effectively already fixed, confirmed this morning, no further action needed for this case.**
- **Page-title gap — `Create_Page_OneOff` specifically (the stale-mapping edge case)**: fix built, Flow Checker clean, published — **but NOT YET TESTED.** No test has actually exercised this action or its new title-setting chain yet.
- To genuinely test this, a scenario needs to be manufactured where: a SharePoint mapping row exists for a given MeetingId, but the OneNote section/page it references has been deleted or is otherwise no longer resolvable as `ExistingMapping`/`ExistingSection`. This is fiddly to set up deliberately and wasn't attempted this session.

## Recommended next steps

1. **Construct the stale-mapping test scenario deliberately**, in a dedicated pass: create a mapping row (or reuse one from an existing test meeting), then delete or rename the OneNote section it points to, then recapture that same MeetingId. Confirm the run routes through `Create_Page_OneOff`, and confirm `Set_PageTitle_OneOff` succeeds and the resulting page gets a real title.
2. Given how narrow this path is, **it may be reasonable to deprioritise this specific test** relative to other pre-presentation priorities (full regression pass across the more common paths), as long as it's clearly flagged as unconfirmed rather than assumed working.
3. Continue tracking the still-open items from the previous handover: the unrelated tail-section anomaly (`Compose SP Item Count` onward), reverting the Bug 9 workaround to genuine title-matching once titles are confirmed reliable everywhere, and the notebook test-section cleanup.

---

**Status: one-off-specific title fix (`Create_Page_OneOff` / `Set_PageTitle_OneOff`) built and published but unconfirmed — narrow edge-case path, not yet exercised by any test. Ordinary one-off captures are already covered by this morning's confirmed recurring-branch fix, since both share the same `Create_OneNote_Page` action for first-time captures. No regression risk identified from today's build — Flow Checker clean throughout, and the untested code path was not previously reachable in a broken state either.**
