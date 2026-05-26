---
name: prep
description: Prepare for an upcoming meeting. Resolves the meeting against today's calendar, pulls attendee context from contacts, memory, and the recurring-meeting register, and prints a one-screen pre-read. Use when the user says "/prep next", "/prep \"team sync\"", "who's in my next meeting", "prep me for the AWS call", or asks for context on an upcoming event. Reads only from the Mission Control workspace; writes nothing.
---

# `/prep`

Pre-meeting context. Reads, prints, does not write.

## Resolution: which meeting?

Argument parsing:

- `/prep next` → first event in `personal/data/calendar.json` whose `start` is strictly in the future, in chronological order. If none, fall back to the next event after now even if today's calendar is done.
- `/prep "<phrase>"` or `/prep <phrase>` → case-insensitive substring match against event `title`, then against keys in `memory/personal/recurring.md` and `personal/inputs/recurring.md`. If multiple match, prefer the one closest to now (future events first, then most recent past).
- `/prep` with no args → equivalent to `/prep next`.

If nothing resolves, print: "No matching event in today's calendar. Try `/prep next` or pass a substring of the meeting title."

## Reads

In this order:

1. `personal/data/calendar.json` — the canonical event data: time, attendees, organizer, location.
2. `personal/inputs/recurring.md` — domain-specific standing-meeting notes (calendar-title match, weekly purpose, what to bring).
3. `memory/personal/recurring.md` — broader recurring-meeting memory.
4. For each attendee email: `personal/inputs/contacts.md` first, then `memory/people.md`. Don't invent context if neither file has the person.

## Writes

None. `/prep` prints to chat only. Never updates calendar, sends invites, or modifies any file.

## Output format

A single chat response, scannable in one screen:

```
{Meeting title}
{Start - End, local}  -  {organizer or "no organizer"}

Why this meeting
{One line from inputs/recurring.md if standing; otherwise inferred from title or empty.}

Attendees ({count})
- {Name or email} - {one-line context from contacts/memory, or "no context on file"}
- ...

What to bring
{From inputs/recurring.md "what to bring" if available; otherwise omit this section.}

Open threads with attendees
{Any recent emails (inbox.json) or Slack mentions (slack.json) from any attendee, with the snippet and link/permalink. Skip if none.}

Notes
{Anything else from inputs/recurring.md "notes" or memory/people.md last-context lines worth surfacing.}
```

If `personal/inputs/contacts.md` is empty for external attendees, say so explicitly rather than padding with generic descriptions: "No external contacts on file yet; add entries to personal/inputs/contacts.md to enrich future preps."

## Voice

Lead with relationship and current arc, not generic title. "Greg runs the AWS Marketplace push from the Greg-Fina-AWS side" beats "Greg Fina, employee at AWS." First-person where natural ("you owe Greg three things from May 21").

No em dashes. No filler ("Here's a summary..."). Print only what changes how the user walks into the meeting.

## Edge cases

- **Attendee resolution overlap:** if a person is in both `contacts.md` and `memory/people.md`, prefer `contacts.md` (it has meeting-specific context).
- **Big invite lists (10+ attendees):** show the top 5 with context; collapse the rest into "and {N} others from {orgs}."
- **Internal-only meeting:** skip "Open threads with attendees" if the meeting is purely Fundamental internal and there are no relevant unread items.
- **Past meeting:** still prep it (sometimes the user wants context after the fact), but lead with a one-line "past, ran at {time}."

## File references

- Spec: `spec/mission-control-spec.md` §4.4
- Calendar data: `personal/data/calendar.json`
- Standing meetings: `personal/inputs/recurring.md`, `memory/personal/recurring.md`
- People: `personal/inputs/contacts.md`, `memory/people.md`
- Cross-reference for open threads: `personal/data/inbox.json`, `personal/data/slack.json`
- Domain voice: `personal/CLAUDE.md`
