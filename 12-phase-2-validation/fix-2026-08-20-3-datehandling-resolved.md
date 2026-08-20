# FIX LOG — #3 date-handling resolved (Topic), FA16 defensive guard pending (20 August 2026)

## Status
**Topic fix built, published, and confirmed working live. Flow A defensive guard (FA16) identified and ready but not yet built.**

## What was fixed
Field observation #3 (date entry) turned out to be two things:
1. The reported "loose formats don't work" premise was mostly false — `20 Aug`/`20 aug` always worked. But `20/08/26` (slash format) genuinely failed, and worse, caused a hard `FlowActionBadGateway` crash rather than a clean rejection.
2. Root cause (confirmed via diagnostics, not assumption): the Topic's `C6C_Check_Date` guard required `DateValue()` to succeed, but `DateValue("20/08/26")` **errors** in this locale (proven via live diagnostic — see below). With no `else` branch on `conditionGroup_BsGPk1`, unrecognised input fell straight through to `invokeFlowAction_bIIKPf`, sending the raw date string as `text_1` into Flow A, where `FA16_Compose_SelectedIndex` crashed calling `int()` on it.

## Evidence gathered before building (diagnostic-first)
Two throwaway diagnostic Topics were pasted in, run via Test panel, and reverted — no guesswork.

**Diagnostic 1 (IsError behaviour):**
- `DateValue("3")` → ERROR (bare number correctly not a date)
- `DateValue("20/08/26")` → ERROR (slash format not parseable by DateValue in this locale — the actual root cause)
- `Value("20/08/26")` → ERROR
- `Value("3")` → NO_ERROR
- Current `C6C` guard on `20/08/26` → **NOT CAUGHT (leaks to Flow A)** — smoking gun
- Proposed simplified guard on `3` → NOT A DATE (number selection safe)

**Diagnostic 2 (full parser, real inputs):**
- Detection: `20/08/26` FIRES, `20 Aug` FIRES, `3` does not fire — all correct
- Parsing: `20/08/26` → 2026-08-20, `20/08/2026` → 2026-08-20, `20 Aug` → 2026-08-20 — all correct, including the 2-digit-year → 2026 normalisation (avoids the `Date()` 1926 trap)

## The fix applied (Topic — 3 changes, diff-verified as the only changes)
Applied via full-Topic YAML import (proven low-risk workflow). Decision: support **both** slash (`20/08/26`, `20/08/2026`) and text (`20 Aug`) formats.

**A1 — `C6C_Check_Date` condition** changed from:
```
=IsError(Value(Topic.TopicSelectedNumber)) && !IsError(DateValue(Topic.TopicSelectedNumber))
```
to:
```
=(!IsError(DateValue(Topic.TopicSelectedNumber)) && IsError(Value(Topic.TopicSelectedNumber))) || IsMatch(Topic.TopicSelectedNumber, "^\d{1,2}/\d{1,2}/\d{2,4}$")
```

**A2 — `C6C` DateContext value** changed from a bare `DateValue()` to a parser that handles both shapes (slash via Split+Date with 2-digit-year normalisation, text via DateValue fallback):
```
=Text(
   If(
     IsMatch(Topic.TopicSelectedNumber, "^\d{1,2}/\d{1,2}/\d{2,4}$"),
     Date(
       If(Len(Last(Split(Topic.TopicSelectedNumber, "/")).Value) <= 2, 2000 + Value(Last(Split(Topic.TopicSelectedNumber, "/")).Value), Value(Last(Split(Topic.TopicSelectedNumber, "/")).Value)),
       Value(Index(Split(Topic.TopicSelectedNumber, "/"), 2).Value),
       Value(First(Split(Topic.TopicSelectedNumber, "/")).Value)
     ),
     DateValue(Topic.TopicSelectedNumber)
   ),
   "yyyy-MM-dd")
```

**A3 — new `elseActions` on `conditionGroup_BsGPk1`** (the safety net — there was none before, which is *why* input leaked to Flow A):
```
elseActions:
  - SendActivity: "Sorry, I didn't recognise that. Type P for previous day, N for next day, or a date like 23 Oct or 23/10/26."
  - GotoAction: question_XFJmje   # re-prompt instead of leaking to Flow A
```

## Live confirmation
Confirmed working end to end via Teams/Test after publish: slash dates, text dates, the previously-crashing three-dates-in-a-row sequence, junk input (now re-prompts cleanly), and normal number selection (unaffected). David confirmed "#3 is working."

## Still pending — FA16 defensive guard (Flow A)
Not yet built. With the Topic fix in place, dates are intercepted upstream and should never reach Flow A as text, so this is defence-in-depth, not the primary fix — deliberately deferred to a separate publish to keep changes isolated (corruption-history caution).

**Ref FA16-FIX** — change `FA16_Compose_SelectedIndex` from:
```
if(equals(trim(variables('varInSelectedNumber')), ''), 0, sub(int(trim(variables('varInSelectedNumber'))), 1))
```
to (digit-strip guard so int() is only called on genuinely-numeric input; returns -1 otherwise, which FA18 SelectedIndexInRange already handles safely):
```
if(and(not(equals(trim(variables('varInSelectedNumber')), '')), empty(replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(trim(variables('varInSelectedNumber')), '0',''),'1',''),'2',''),'3',''),'4',''),'5',''),'6',''),'7',''),'8',''),'9',''))), sub(int(trim(variables('varInSelectedNumber'))), 1), -1)
```
Note: an earlier draft of this guard called `int()` inside the guard itself, which would have crashed on the very input it guards against — corrected to the digit-strip approach before recommending.

## Repo housekeeping
- Current Topic YAML with this fix should be re-exported and committed as `topic-export-2026-08-20-fix3.yaml` next time it's convenient (the pre-fix `topic-export-2026-07-31.yaml` remains the last committed version; note it no longer matches the live Topic).
- The `known-good-values-master-reference.md` covers Flow B only; once FA16-FIX lands, Flow A's key expressions should get the same treatment.

---
*Confirmed 20 August 2026 via two live throwaway diagnostics + post-fix live test. Supersedes the diagnosis-only `bug-2026-08-20-fa16-int-crash-date-leak.md` for the Topic portion; that doc's FA16 (Flow A) recommendation remains open until FA16-FIX is built.*
