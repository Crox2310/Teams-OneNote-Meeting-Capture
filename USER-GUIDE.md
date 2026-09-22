# User guide — Meeting Capture

How to use the Meeting Capture agent in Teams, and what to expect on the OneNote page.

---

## Capturing a meeting

1. Open a chat with the Meeting Capture agent in Teams and ask it to capture a meeting.
2. The agent shows today's meetings in time order. Holidays, leave and all-day entries are left out.
3. If today isn't the day you want:
   - **P** — previous day
   - **N** — next day
   - type a date to jump straight to it
   - cancel to stop
4. Pick the meeting. If there is only one that day, the agent may confirm it with you rather than list it.
5. The agent replies with a link to the OneNote page.

You can do this before, during or after the meeting. Capturing ahead of time means the page is ready for your own notes in the meeting.

**Tip:** start each capture in a new chat with the agent. Old conversations can hold on to earlier choices.

---

## Where the page goes

| Meeting | Section | Page title |
|---|---|---|
| Recurring | Its own section, one per series (`Mtg - <meeting name>`) | `<meeting name> - <date>` |
| One-off | **One-Off Meetings**, inside the **One Off Meeting** section group | `<meeting name> - <date>` |

Each occurrence of a recurring meeting gets its own page. Capturing the same occurrence again reuses the existing page instead of creating a second one.

Section names are shortened if the meeting name is long.

---

## What's on the page

| Section | Filled by | When |
|---|---|---|
| Join Teams meeting link | Set-up | Straight away (only if the invite has a Teams link) |
| **My Notes** | You | Any time |
| **Meeting Notes** | Automatic | About 30 minutes after the meeting ends, if Teams produced a Copilot recap |
| **Chat Transcript** | Automatic | About 30 minutes after the meeting ends |
| **Meeting Capture** | Set-up | Straight away: the invite's agenda or description, without the Teams dial-in text |

Write your own notes under **My Notes**. The automatic sections only ever add content; they never change what you've written.

---

## What to expect after the meeting

- The fill runs once per meeting. If it ran before the Copilot recap was ready, Meeting Notes stays empty and only the chat is added.
- No recap appears if the meeting wasn't recorded or transcribed, or Copilot wasn't used.
- The fill needs a Teams meeting. In-person meetings without a Teams link get a page but no recap or chat.
- Only meetings from today and yesterday are picked up automatically. If you capture an older meeting, its page is created but not filled.

If a page hasn't filled an hour after the meeting ended, see `RUNBOOK.md` → "A page didn't fill".

---

## Reporting a problem

Add a row to the **Field-testing intake** table in `BACKLOG.md`: the date, the meeting type (recurring or one-off), what happened, what you expected, and a run ID or screenshot if you have one.
