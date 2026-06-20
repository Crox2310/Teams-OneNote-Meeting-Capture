# Living Audit — Topic Orchestration (Copilot Studio)

Companion document to `living-audit.md` (Flow A / Flow B expression-level bugs). This document covers the **Copilot Studio Topic layer** — orchestration logic deciding which flow gets called, with what inputs, and how results route back to the conversation.

See `PROCESS-expression-audit-maintenance.md` for the shared maintenance discipline. Update this the moment Topic structure changes, before closing out the session's handover note.

**Last updated:** 2026-06-21 (full Topic YAML + Designer screenshots reviewed; supersedes the earlier skeleton draft of this file)
**Topic:** "Meeting Capture (v4 rebuild)" — published 6/20/2026, full source YAML now on file
**Status key:** 🔴 confirmed bug · 🟡 suspect/unconfirmed · 🟢 confirmed working, tested · ⚪ confirmed clean/by design · 🔵 designed but not yet live-tested

---

## 1. Purpose and scope of this document

Where `living-audit.md` answers "is this expression correct," this document answers: what does the Topic do end to end, which nodes call which flow with which inputs, what are the branch conditions, what does the user see, and what's confirmed-working vs designed-but-unverified.

---

## 2. Trigger

🟢 **`User says a phrase`** — phrase-triggered, not adaptive-card or message-extension based. Phrases: `capture meeting notes`, `capture notes for`, `record meeting notes`, `create meeting notes`, `capture`. Note `capture` alone is a very broad trigger phrase — worth being aware this could over-fire on unrelated messages containing that word, though not confirmed as an actual problem.

---

## 3. Stage 1 — Initial meeting resolution (`C2_Call_FlowA_Initial`)

🟢 Calls Flow A (`PA - Resolve Meeting Selection - v1 Clean Build`, flowId `d9d7ccf7-7d61-f111-a826-6045bde03856`).

**Inputs (Topic → Flow A), confirmed verbatim from YAML:**
| Flow A field | Source |
|---|---|
| UserSearchText | `System.Activity.Text` |
| InSelectedNumber | `" "` (literal space — first call, no selection made yet) |
| OriginalUserSearchText | `System.Activity.Text` |
| DateContext | `today` (literal) |
| MaxCandidates | `5` (literal) |

**Outputs (Flow A → Topic), confirmed verbatim:**
| Flow A output | Topic variable |
|---|---|
| status | Topic.Status |
| matchcount | Topic.MatchCount |
| candidatelist | Topic.CandidateList |
| meetingtitle | Topic.MeetingTitle |
| calendareventid | Topic.CalendarEventId |
| isrecurring | Topic.IsRecurring |
| seriesmasterid | Topic.SeriesMasterId |

⚪ This input/output contract is clean and fully documented — no ambiguity, unlike the Flow B trigger (see Section 5).

---

## 4. Three-way branch on `Topic.MatchCount` (`conditionGroup_k0eXvc`, displayed as `C4_Check_MatchCount`)

Confirmed structure from YAML — this **is** the three-way split referenced in prior session handovers as "drafted, not yet published." It is now visibly live in the published Topic (Published 6/20/2026 banner in Designer), so it appears to have been published since that note was written — worth confirming explicitly with David whether that publish was intentional or incidental.

### Branch 1 — NO_MATCH (`Topic.MatchCount = Text(0)`)
🟢 — straightforward equality check, no guard issues.
**Response (`C4A_NoMatch_Message`):** *"I couldn't find a meeting matching that for today. Could you try a different name, or check the date?"*

### Branch 2 — Single match (`Topic.MatchCount = Text(1)`, displayed as `C8_Check_SingleMatch`)
🔴 **Flow reference broken** — see Section 5.
1. `C8A_Confirm_SelectedMessage`: *"Your meeting {Topic.MeetingTitle} has been found. Creating your OneNote page now..."*
2. `C8B_Call_FlowB_Create_Page` — calls Flow B. **Confirmed broken in Designer: "Flow not found or is turned off"** (image 3, 2026-06-21).
3. `C8C_Check_OutStatus` (`Topic.OutStatus = "OK"`) — **confirmed broken: "Incompatible type comparison. Type: String, expected: Unspecified"** (image 3). This strongly suggests `Topic.OutStatus` is not currently typed as a string — likely a knock-on effect of the C8B flow reference being broken, since the output binding can't resolve its type without a working flow connection.
   - True → `C8D_Success`: *"Great news! Your meeting notes for {Topic.MeetingTitle} have been saved to OneNote. Here's your page link: {Topic.OutCreatedPageLink}"*
   - False/else → `C8E_Error`: *"I'm sorry, something went wrong saving your meeting notes. Please try again."*

### Branch 3 — Multi-match (`elseActions` on the C4 condition group, displayed as "All other conditions")
🔴 **Flow reference broken** — see Section 5. Otherwise designed but never live-tested (UJ2 outstanding).
1. `C5_Display_CandidateList`: shows `{Topic.CandidateList}` (raw dynamic content — exact list formatting determined by Flow A's `candidatelist` output, not by Topic-level text).
2. `C6_Ask_SelectedNumber`: *"Which one? Enter a number."* — captures user's entire response into `Topic.TopicSelectedNumber` (StringPrebuiltEntity, no validation that it's actually numeric).
3. `C7_Call_FlowA_Selection` — calls Flow A again, this time with the user's selection:
   | Flow A field | Source |
   |---|---|
   | UserSearchText | `System.Activity.Text` |
   | InSelectedNumber | `Topic.TopicSelectedNumber` |
   | OriginalUserSearchText | `System.Activity.Text` |
   | DateContext | `today` |
   | MaxCandidates | `5` |

   Same output binding as `C2_Call_FlowA_Initial` (overwrites the same Topic variables).
4. `C9_Confirm_SelectedMeeting`: *"Great — I've found your meeting: {Topic.MeetingTitle}"*
5. `C10_Call_FlowB_Create_Page` — calls Flow B. **Confirmed broken in Designer: "Flow not found or is turned off"** (images 4-5, 2026-06-21).
6. `C11_Check_OutStatus` (`Topic.OutStatus = "OK"`) — **same incompatible-type warning as C8C** (image 5).
   - True → `C12_Success`: identical message text to `C8D_Success`.
   - False/else → `C12_Error`: identical message text to `C8E_Error`.

🟡 **Design note, not a bug:** `OriginalUserSearchText` is bound to `System.Activity.Text` in both C2 and C7 calls — i.e. it's re-read from the *current* message each time, not preserved from the original first message. In the multi-match flow, by the time C7 fires the user has typed a number (e.g. "2"), not their original search text. Worth checking whether Flow A actually uses `OriginalUserSearchText` for anything in the selection path, since if it does, it may now be receiving "2" instead of the original meeting name — this could be silently degrading match quality on the second call. Not confirmed as a live bug, but flagged for investigation.

---

## 5. Stage 2 — OneNote page creation (Flow B calls: `C8B_Call_FlowB_Create_Page` and `C10_Call_FlowB_Create_Page`)

🔴 **Both confirmed broken in Designer as of 2026-06-21** — "Flow not found or is turned off" on both nodes. This is a live, user-facing blocker: neither the single-match nor the multi-match path can currently create a OneNote page at all, regardless of any Flow B internal expression fixes. This needs investigating before any of the `living-audit.md` Flow B fixes will be testable end-to-end. Possible causes worth checking next session: Flow B's flowId changed (republish creating a new ID rather than updating in place), the flow was accidentally turned off, or a connection reference issue similar to the SharePoint/Outlook/OneNote consent-loop incident from 2026-06-20.

**Both nodes are otherwise structurally identical** — confirmed verbatim from YAML, this is the resolution to `living-audit.md`'s open trigger-mapping question:

| Flow B trigger field (display title) | Topic source |
|---|---|
| `text` (IsRecurring) | `Topic.IsRecurring` |
| `text_1` (MeetingTitle) | `Topic.MeetingTitle` |
| `text_2` (SeriesMaster) | `Topic.SeriesMasterId` |
| `text_3` (PageHtml) | `Concatenate("<h1>", Topic.MeetingTitle, "</h1><p>Meeting notes captured via Teams OneNote Meeting Capture agent.</p>")` |
| `text_4` (MeetingId) | `Topic.CalendarEventId` |

🟢 **This confirms the fix for `living-audit.md`'s `Condition_IsRecurring` bug**: Flow B should read `triggerBody()?['text']`, not `triggerBody()?['IsRecurring']`. No longer needs separate verification — apply this fix with confidence next session.

🟡 **Unconfirmed pairing, newly spotted this pass:** `text_4`'s display title is "MeetingId" but the Topic supplies `Topic.CalendarEventId`. These may be the same identifier space (Outlook calendar event ID used as the meeting ID throughout), but that equivalence has not been explicitly confirmed anywhere in either `living-audit.md` or this document. Worth a quick verification next session — if they're different ID schemes, this could be a quiet correctness bug independent of the broken-flow-reference issue above.

**Outputs (Flow B → Topic), confirmed verbatim — identical binding on both C8B and C10:**
| Flow B output | Topic variable |
|---|---|
| outstatus | Topic.OutStatus |
| outpageaction | Topic.OutPageAction |
| outcreatedpagelink | Topic.OutCreatedPageLink |
| outcreatedpageselfurl | Topic.OutCreatedPageSelfUrl |
| outexistingpageselfurl | Topic.OutExistingPageSelfUrl |
| outagentresponsesummary | Topic.OutAgentResponseSummary |
| outbranchresult | Topic.OutBranchResult |
| outfinaltargetsectionpagesurl | Topic.OutFinalTargetSectionPagesUrl |
| outisrecurring | Topic.OutIsRecurring |
| outmatchcount | Topic.OutMatchCount |
| outmeetingtitle | Topic.OutMeetingTitle |
| outonenoteresolverresult | Topic.OutOneNoteResolverResult |
| outpagedecision | Topic.OutPageDecision |
| outpagehtml | Topic.OutPageHtml |
| outpageroute | Topic.OutPageRoute |
| outresolverresult | Topic.OutResolverResult |
| outseriesmasterid | Topic.OutSeriesMasterId |
| outspitemcount | Topic.OutSPItemCount |
| outtargetsectionpagesurl | Topic.OutTargetSectionPagesUrl |
| outupdatehtmlfragment | Topic.OutUpdateHtmlFragment |

⚪ This is a much larger output set than `living-audit.md`'s "Respond to the agent" section had fully itemized (that document trailed off with "additional outputs below the fold not yet re-confirmed"). This table is now the complete, confirmed list — `living-audit.md` should be updated to reference this table rather than duplicating it, next time that file is touched.

Of these, **only `Topic.OutStatus` and `Topic.OutCreatedPageLink` are actually consumed** by the Topic's response logic (Section 4, C8C/C8D/C8E and C11/C12). The rest are captured into Topic variables but not currently used in any visible branch or message — either dead/reserved for future use, or used elsewhere in the Topic not covered by this YAML excerpt. Worth a quick check next session on whether that's intentional.

---

## 6. Topic-to-flow contract summary

This supersedes Section 4's placeholder table from the earlier skeleton draft of this document.

| Direction | Status |
|---|---|
| Topic → Flow A (both calls) | ⚪ fully documented, fields confirmed clean |
| Flow A → Topic (both calls) | ⚪ fully documented, fields confirmed clean |
| Topic → Flow B (both calls) | ⚪ fully documented; resolves `living-audit.md`'s trigger-mapping open item |
| Flow B → Topic (both calls) | ⚪ fully documented (19 outputs); only 2 currently consumed downstream |
| **Flow B connectivity itself** | 🔴 **broken on both call nodes** — this is now the top-priority item across both documents |

---

## 7. Live-test status by branch

| Branch | Status | Last verified |
|---|---|---|
| NO_MATCH | 🟢 logic confirmed clean; live response text confirmed via YAML | Not separately live-tested this pass, but no blockers |
| Single match (UJ1) | 🔴 was working 2026-06-20 (pre-dates this session's discovery of the broken C8B flow reference) — **status now uncertain pending investigation** | 2026-06-20 (since superseded by the broken-flow-reference finding) |
| Multi-match (UJ2) | 🔴 blocked on broken C10 flow reference; also never exercised even before that | Not yet tested this project |
| Recurring, new series (UJ3) | 🔴 blocked — both the Flow B trigger-key bug (now resolved, fix not applied) and the broken flow reference | — |
| Recurring, existing series (UJ4) | 🔴 same blockers as UJ3 | — |

**Priority shift from this session's findings:** the broken C8B/C10 flow references are now the single highest-priority item — they block every branch except NO_MATCH, and supersede the importance of the individual Flow B expression fixes catalogued in `living-audit.md`, since none of those fixes can be live-tested until the flow reference itself resolves.

---

## 8. Open items / not yet covered by this document

- **Urgent:** diagnose why C8B and C10 both show "Flow not found or is turned off." Check: has Flow B's flowId changed since last publish? Is the flow turned on in Power Automate? Is this connection-reference related (cf. the 2026-06-20 evening consent-loop incident)?
- **Urgent:** diagnose the `Incompatible type comparison. Type: String, expected: Unspecified` warning on `C8C_Check_OutStatus` / `C11_Check_OutStatus` — likely a symptom of the above, but worth confirming it self-resolves once the flow reference is fixed rather than being a separate issue.
- Confirm whether `Topic.CalendarEventId` and Flow B's "MeetingId" trigger field are genuinely the same identifier space.
- Confirm whether `OriginalUserSearchText` being re-read from `System.Activity.Text` on the second Flow A call (C7) is intentional or a quiet bug affecting multi-match selection quality.
- Confirm whether the three-way split being visibly published (6/20/2026) was intentional, given prior session notes described it as "drafted, not yet published."
- Confirm intentional use (or lack thereof) of the 17 Flow B output variables not currently consumed by any Topic branch.
- UJ5 — not yet covered in any document.

---

## 9. Pointer for future Claude instances

Read `PROCESS-expression-audit-maintenance.md` first, then this document for Topic-level design intent and current live-test status, then `living-audit.md` for flow-level expression detail. As of this update, the single most important fact across both documents is that the Flow B connector itself appears broken at the Topic level — start there before resuming any expression-level fix work.
