# Flow B (PA - Resolve OneNote Meeting Section) — Back-Half Peek-Code Trace

**Date:** 2026-07-18
**Method:** Full peek-code trace of Flow B from `Condition Should Create Page` through to `Respond to the agent`, action by action, cross-checked against `living-audit.md`'s secondhand description.
**Status:** No crash-causing bugs found. Three findings worth tracking (below), plus one still-unverified section identified.

---

## 1. Structural map (now confirmed via peek code)

```
Condition Should Create Page  (runs after Condition_Mapping_Exists: Succeeded)
├─ True — brand-new page
│  ├─ Create OneNote Page          (connection: shared_onenote; notebookKey: "Meeting Notes|$|<SharePoint URL>";
│  │                                 sectionId: variables('varTargetSectionPagesUrl'); pageContent: triggerBody()?['text_3'])
│  ├─ Compose PageSelfUrl Created  (body('Create_OneNote_Page')?['self'])
│  ├─ HTTP Update SP PageSelfUrl   (POST to RecurringMeetingSectionMap list, X-HTTP-Method: MERGE,
│  │                                 keyed by Filter_Existing_Mapping's ID or the earlier SharePoint request's ID)
│  ├─ Set varPageAction = "Created"
│  └─ Set varOutputPageSelfUrl Created
│
└─ False — page may already exist, or one-off mode
   └─ Condition Is Genuine Existing Page
      ├─ True — existing page confirmed
      │  ├─ Get Sections Existing Branch
      │  ├─ Filter Existing Section By Name
      │  ├─ Apply to each Existing Section
      │  │  └─ Update page content Existing Branch  (append content after existing body, via Compose_UpdateHtmlFragment)
      │  ├─ Set varPageAction = "UpdatedAppend"
      │  └─ Set varOutputPageLink Existing  (from Compose_ExistingPageSelfUrl)
      │
      └─ False (else) — no genuine existing page found
         ├─ Create_Page_OneOff  (connection: shared_onenote-1 — different connection reference than above;
         │                        same notebookKey/sectionId pattern, pageContent: triggerBody()?['text_3'])
         └─ Set_varOutputPageLink_Created_OneOff  (from Create_Page_OneOff's oneNoteWebUrl)

↓ (both branches merge here)

Compose AgentResponseSummary   — builds a human-readable sentence keyed off varPageAction (Created / UpdatedAppend / ExistsNoCreate / fallback), quoting triggerBody()?['text_1'] (MeetingTitle)
Compose SP Item Count          — length(body('Get_items')?['value'])  ← 'Get_items' not yet traced, lives upstream
Set varOutStatus = "OK"        — hardcoded, no branch coalescing (see Finding 1)
Respond to the agent           — 20-field response schema, all populated (see Section 2)
```

This matches `living-audit.md`'s secondhand description of this section — no structural surprises like the FA09A or C6B_Check_N discoveries. The design (separate "create new," "append to existing," and "one-off create" paths, converging on a single summary/status/respond block) is sound and matches the intended behaviour described in the project's own documentation.

---

## 2. Respond to the agent — full output mapping

| Field | Source |
|---|---|
| `outisrecurring` | `triggerBody()?['text']` (passthrough) |
| `outmeetingtitle` | `triggerBody()?['text_1']` (passthrough) |
| `outseriesmasterid` | `triggerBody()?['text_2']` (passthrough) |
| `outpagehtml` | `triggerBody()?['text_3']` (passthrough) |
| `outspitemcount` | `int(coalesce(outputs('Compose_SP_Item_Count'), 0))` |
| `outmatchcount` | `variables('varFinalMatchCount')` |
| `outbranchresult` | `if(greater(int(coalesce(outputs('Compose_Match_Count'), 0)), 0), 'EXISTS', 'CREATE_REQUIRED')` |
| `outonenoteresolverresult` | `variables('varOneNoteResolverResult')` |
| `outtargetsectionpagesurl` | `variables('varTargetSectionPagesUrl')` |
| `outcreatedpagelink` | `variables('varOutputPageLink')` |
| `outcreatedpageselfurl` | `variables('varOutputPageSelfUrl')` |
| `outfinaltargetsectionpagesurl` | `variables('varTargetSectionPagesUrl')` — **identical source to outtargetsectionpagesurl** |
| `outresolverresult` | `variables('varOneNoteResolverResult')` — **identical source to outonenoteresolverresult** |
| `outexistingpageselfurl` | `variables('varFinalExistingPageSelfUrl')` |
| `outpagedecision` | `variables('varFinalPageDecision')` |
| `outpageroute` | `if(equals(outputs('Compose_PageDecision'), 'PAGE_EXISTS'), 'PAGE_EXISTS_ROUTE', 'PAGE_NOT_FOUND_ROUTE')` |
| `outpageaction` | `variables('varPageAction')` |
| `outupdatehtmlfragment` | `outputs('Compose_UpdateHtmlFragment')` |
| `outagentresponsesummary` | `outputs('Compose_AgentResponseSummary')` |
| `outstatus` | `variables('varOutStatus')` — always `"OK"` (see Finding 1) |

---

## 3. Findings

### Finding 1 🟡 — `OutStatus` has no error path; always returns "OK"
Unlike Flow A's `Status` field (coalesced across NoMatch/Multi/Error/Resolved branches — confirmed clean in the 2026-07-16 audit), Flow B's `varOutStatus` is set by a single unconditional `SetVariable` action (`value: "OK"`) with no alternate branch. If any upstream action in this flow throws (e.g. `Create_Page_OneOff` fails due to an invalid `sectionId`, or the SharePoint MERGE request fails), the flow run would error out at the platform level rather than gracefully returning `OutStatus: "Error"` to the Topic. The Topic's own error handling (`C11_Check_OutStatus` checking `Topic.OutStatus = "OK"`) would never see this — it would only ever see a hard flow failure instead. Not currently causing any known symptom, but worth hardening to match Flow A's pattern.

### Finding 2 🟡 — Two different OneNote connections used for page creation
`Create OneNote Page` (True/brand-new-page branch) uses connection `shared_onenote`. `Create_Page_OneOff` (False/one-off branch) uses connection `shared_onenote-1`. Both point at the same notebook/section pattern, so this is very likely just two separate connection references configured in the same environment rather than a functional bug — but if the two connections ever diverge in scope or get individually revoked, one branch could silently start failing while the other keeps working. Worth standardising on a single connection reference when convenient.

### Finding 3 ⚪ — Duplicate output fields (cosmetic, not a bug)
`outtargetsectionpagesurl` / `outfinaltargetsectionpagesurl` and `outresolverresult` / `outonenoteresolverresult` are each pairs of fields sourced from the exact same variable. Harmless — likely legacy/redundant naming from an earlier iteration — but worth knowing they'll always match each other exactly, so there's no need to treat them as independent signals.

---

## 4. Remaining gap

`Condition Mapping Exists` (and whatever precedes it in the flow) was **not** included in this trace — the screenshots started at `Condition Should Create Page`. This upstream section is where `Get_items`, `Compose_Match_Count`, `varTargetSectionPagesUrl`, `varOneNoteResolverResult`, `varFinalMatchCount`, `varFinalPageDecision`, `varFinalExistingPageSelfUrl`, and `Compose_PageDecision` are all first assigned — everything this back half consumes. This is genuinely the last unverified piece of Flow B and arguably the most important, since it's where the SharePoint recurring-meeting-to-section mapping lookup actually happens. Recommend tracing this next before considering Flow B fully audited.

---

## 5. Open items / not yet covered

- `Condition Mapping Exists` and upstream mapping-lookup logic — not yet traced (see Section 4).
- Finding 1 (OutStatus hardening) and Finding 2 (connection standardisation) — flagged, not fixed. Neither is causing a known live issue, so safe to prioritise behind anything actively broken.
- FA43 `IsRecurring`/`SeriesMaster` coalescing gap (Flow A, Finding 4 from 2026-07-16 doc) — still open, unrelated to this trace.
