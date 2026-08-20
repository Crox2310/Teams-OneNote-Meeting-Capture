# Flow reference — full container-level Peek Code capture (20 August 2026)

## Purpose

This is the most current full structural capture of **Flow B** (`PA - Resolve OneNote Meeting Section - v2 Clean Build`), captured via **container-level Peek Code** — selecting Peek Code on a top-level container (e.g. a `Condition`) captures every nested action inside it in one go, rather than clicking through actions individually. This is a much faster capture method discovered this session and should be the default approach going forward.

This supersedes `flow-reference-2026-08-15-full-peek-code-capture.md` as the current reference — that capture predates the Bug 8 fix, Incident 7 recovery, and the append-fragment content-loss bug found this session (`bug-2026-08-20-update-fragment-discards-new-content.md`). The 15 August doc remains useful as a historical record of the corruption-recovery process, but should not be treated as current flow state.

**Not captured this session**: `Condition_IsRecurring`'s own container (only its children were captured via `Condition Mapping Exists` and `Condition Should Create Page`), and the full `Filter_Pages_By_Title` / `Compose_RealExistingPageId` internals were captured but not independently re-verified against Designer since 16 August — see `bug-2026-08-20-update-fragment-discards-new-content.md` and `design-amendment-2026-08-20-per-occurrence-recurring-pages.md` for what's changed since.

---

## Trigger — `When an agent calls the flow`

```json
{
  "type": "Request",
  "kind": "Skills",
  "inputs": {
    "schema": {
      "type": "object",
      "properties": {
        "text_1": { "title": "MeetingTitle", "type": "string" },
        "text_2": { "title": "SeriesMasterId", "type": "string" },
        "text_3": { "title": "PageHtml", "type": "string" },
        "text_4": { "title": "MeetingId", "type": "string" },
        "text": { "title": "IsRecurring", "type": "string" }
      },
      "required": ["text_1", "text_2", "text_3", "text_4", "text"]
    }
  }
}
```

Five inputs only. No date/occurrence field exists as its own input — the occurrence date is only available embedded in `text_3`'s `<title>` tag (see `design-amendment-2026-08-20-per-occurrence-recurring-pages.md`).

## `Get items` (SharePoint)

```json
{
  "type": "OpenApiConnection",
  "inputs": {
    "parameters": {
      "dataset": "https://jsainsbury.sharepoint.com/sites/coplt",
      "table": "186b3c9f-e758-4e85-83d5-685946614a0a"
    },
    "host": { "apiId": "/providers/Microsoft.PowerApps/apis/shared_sharepointonline", "connection": "shared_sharepointonline", "operationId": "GetItems" }
  }
}
```

## InitializeVariable chain (top of flow)

In order: `varFinalExistingPageSelfUrl`, `varFinalPageDecision`, `varFinalMatchCount`, `varOutStatus`, `varOutputPageLink`, `varOutputPageSelfUrl`, `varTargetSectionPagesUrl`, `varOneNoteResolverResult`, `varPageAction` — all plain string `InitializeVariable` actions, no values, chained via `runAfter`.

---

## `Condition Mapping Exists` (top-level container)

Gates the recurring vs one-off split.

**TRUE branch (recurring path)**:
- `Compose_Input_SeriesMasterId`: `@triggerBody()?['text_2']`
- `Compose_Input_MeetingTitle`: `@triggerBody()?['text_1']`
- `Filter_Existing_Mapping`: SharePoint `Get_items` filtered on `item()?['SeriesMasterId'] == triggerBody()?['text_2']`
- `Compose_ExistingPageSelfUrl`: first match's `PageSelfUrl`, or `''`
- `Compose_PageDecision`: `PAGE_EXISTS` / `PAGE_NOT_FOUND`
- `Compose_Match_Count`: `length(...)`
- `varFinalExistingPageSelfUrl_1` / `varFinalPageDecision_1` / `varFinalMatchCount_1`: SetVariable from the above three Composes

**FALSE branch (one-off path)**:
- `FB-F01 — Compose Input MeetingTitle (one-off)`: sanitises `text_1` into a safe section name, falls back to `'Mtg - Untitled Meeting'` if empty
- `Get_Sections_OneOff` → `Filter_OneNote_Section_OneOff` → `Compose_Section_Match_Count_OneOff` → `Condition_Section_Exists_OneOff`
  - TRUE: `For_each_1` sets `varTargetSectionPagesUrl` / `varOneNoteResolverResult = 'ExistingSection'`
  - FALSE: `Create_Section_OneOff` then sets the same two vars, `varOneNoteResolverResult = 'CreatedSection'`
- `OF01 — Filter_Existing_Mapping_OneOff`: filtered on `item()?['MeetingId'] == triggerBody()?['text_4']` — **this is the one-off equivalent of the recurring branch's `SeriesMasterId` filter, keyed on `MeetingId` instead**
- `OF02`–`OF04`: same Compose pattern as recurring (`ExistingPageSelfUrl`, `PageDecision`, `Match_Count`)
- `OF05a`–`OF05c`: SetVariable into the same three shared `varFinal*` variables

**Key structural point**: both branches converge on the same three variables (`varFinalExistingPageSelfUrl`, `varFinalPageDecision`, `varFinalMatchCount`), which is what lets `Condition Should Create Page` (below) treat recurring and one-off uniformly downstream.

---

## `Condition_IsRecurring`'s post-branch actions (captured via `Condition Mapping Exists`'s sibling container)

**TRUE (recurring)**:
- `Compose_Branch_Result`: `'EXISTS'` or (else-branch) `'CREATE_REQUIRED'`
- `Condition_Recurring_TargetSection`: if `IsRecurring == 'true'`, sets `varTargetSectionPagesUrl` from `first(body('Filter_Existing_Mapping'))?['SectionPagesUrl']`
- Else branch: `Condition_Should_Write_Mapping` → `Send_an_HTTP_request_to_SharePoint` (POST new mapping row) if genuinely no match; `Compose_SafeSectionName` (same 43-char sanitisation pattern as one-off); `Get_Sections_Recurring` → `Filter_OneNote_Section_Recurring` → `Condition_Section_Exists_Recurring` (mirrors one-off's exists/create split exactly)

---

## `Condition Should Create Page` (top-level container — the big one)

Expression: `variables('varFinalPageDecision') == 'PAGE_NOT_FOUND'`

**TRUE (create new page)**:
- `Compose_SafePageTitle`: sanitises `text_1`, falls back to `'Untitled Meeting'`, truncated to 150 chars
- `Create_OneNote_Page`: `sectionId: @variables('varTargetSectionPagesUrl')`, `pageContent: <p>@{triggerBody()?['text_3']}</p>` — **note: wraps the full `text_3` HTML document (which already contains its own `<html>` tags) inside another `<p>` tag; flagged as a possible double-wrap issue in `bug-2026-08-20-update-fragment-discards-new-content.md`**
- `Delay_Post_Page_Creation` (5s wait) → `Compose_PageSelfUrl_Created` → `Get_Pages_In_Section_Recurring_PostCreate` → `Filter_Pages_By_SelfUrl_Recurring` → `Compose_ConfirmedCreatedPageId` → `Set_PageTitle_Recurring` (explicit title-set, the 16 August fix)
- `OF09-Gate — Condition Is Recurring (SP Write)`: writes/updates the SharePoint mapping row (`PageSelfUrl`, `PageWebUrl`) for either branch; one-off else-branch has its own near-identical `OF09a`/`OF09b` sequence plus its own title-set/delay/confirm chain

**FALSE (existing page — update path)**:
- `Set_varPageAction_ExistsNoCreate` → `Set_varOutputPageSelfUrl_Existing`
- `Compose_UpdateHtmlFragment`: **hardcoded static string**, does not reference `text_3` — see `bug-2026-08-20-update-fragment-discards-new-content.md` for the confirmed bug this causes
- `Compose_ExistingPageId`: `last(split(variables('varOutputPageSelfUrl'), '/'))`
- `Condition_Is_Genuine_Existing_Page`: `contains(createArray('ExistingMapping','ExistingSection'), variables('varOneNoteResolverResult'))`
  - TRUE: `Get_Sections_Existing_Branch` → `Filter_Existing_Section_By_Name` → `Apply_to_each_Existing_Section` containing `Update_page_content_Existing_Branch` (appends `Compose_UpdateHtmlFragment`'s output, `action: append, position: after`), `Get_Pages_In_Section_Existing_Branch`, `Compose_MeetingTitleForPageMatch`, `Filter_Pages_By_Title` (present but **its output does not appear to feed `Compose_RealExistingPageId`** — see below), `Compose_RealExistingPageId`
    - `Compose_RealExistingPageId`: `@if(greater(length(outputs('Get_Pages_In_Section_Existing_Branch')?['body']?['value']), 0), first(outputs('Get_Pages_In_Section_Existing_Branch')?['body']?['value'])?['id'], '')` — **this is the Bug 9 "first page in section" workaround, confirmed still in place, still a hard blocker for the per-occurrence-pages amendment**
  - FALSE: `Create_Page_OneOff` (this is where Bug 5 originally surfaced) with its own title-set/delay/confirm chain, mirroring the create-new-page branch above

---

## `Response action` (final `Respond to the agent`)

Full 19-field output schema (`outisrecurring`, `outmeetingtitle`, `outseriesmasterid`, `outpagehtml`, `outspitemcount`, `outmatchcount`, `outbranchresult`, `outonenoteresolverresult`, `outtargetsectionpagesurl`, `outcreatedpagelink`, `outcreatedpageselfurl`, `outfinaltargetsectionpagesurl`, `outresolverresult`, `outexistingpageselfurl`, `outpagedecision`, `outpageroute`, `outpageaction`, `outupdatehtmlfragment`, `outagentresponsesummary`, `outstatus`) — all mapped directly from the corresponding variables/Compose outputs described above. `outstatus` still hardcoded to `'OK'` via `Set_varOutStatus` immediately before this action (the still-open OutStatus gap from the 20 July analysis).

---

## Cross-reference

- Bug/design docs arising directly from this capture: `bug-2026-08-20-update-fragment-discards-new-content.md`, `bug-2026-08-20-fa16-int-crash-date-leak.md` (Flow A, not this capture, but found same session), `design-amendment-2026-08-20-per-occurrence-recurring-pages.md`
- Superseded reference: `flow-reference-2026-08-15-full-peek-code-capture.md` (pre-corruption-recovery, pre-page-title-fix state — historical only)
- Capture method note: container-level Peek Code (select the top-level `Condition`, not individual actions) captures all nested actions in one screenshot/paste — much faster than the action-by-action approach used on 15 August. Recommend this as the default capture method for future sessions.

---
*Compiled 20 August 2026 from container-level Peek Code pasted directly into chat during this session, covering `Condition Mapping Exists`, `Condition Should Create Page`, the trigger `Request` action, the `Response` action, and the `InitializeVariable`/`Compose_AgentResponseSummary`/`Compose_SP_Item_Count`/`Set_varOutStatus` chain.*
