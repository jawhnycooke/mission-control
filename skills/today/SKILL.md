---
name: today
description: Re-run today's morning brief on demand, or append a follow-up to the active task list. Use when the user says "/today", "refresh my brief", "rerun the brief", "regenerate the dashboard", or types `/today add "<item>"` to capture a new follow-up. Reads and writes only the personal-morning-brief files in the Mission Control workspace (~/cowork/).
---

# `/today`

On-demand counterpart to the `personal-morning-brief` scheduled task. Two modes.

## Mode 1 — bare `/today`: refresh

Re-run the full `personal-morning-brief` workflow, regardless of when the scheduled task last ran. Same steps, same files, same rules as `spec/mission-control-spec.md` §4.1.

1. Refresh `personal/data/calendar.json`, `inbox.json`, `slack.json` from Calendar, Gmail, Slack. Overwrite each file. If a connector fails, keep the prior good file and surface the gap downstream.
2. Read `~/cowork/TASKS.md` to source follow-ups (Active section).
3. Read `memory/personal/recurring.md` to resolve standing meetings on today's calendar.
4. Regenerate `personal/outputs/dashboard.html` — self-contained dark-mode HTML, no external scripts, every §4.2 section rendered.
5. If `briefs/brief-YYYY-MM-DD.md` exists for a prior day at the top level, move it to `briefs/archive/`. Then write today's `briefs/brief-{YYYY-MM-DD}.md` with all §4.1 sections in first-person chief-of-staff voice.
6. Print a one-line summary: counts, gaps, and the brief + dashboard paths.

**Critical:** never write to `personal/inputs/`. Never send email, Slack, or invites. Read-only against connectors; write-only against `data/`, `outputs/`, and `briefs/`.

## Mode 2 — `/today add "<item>"`: capture a follow-up

Append the item to `TASKS.md` Active. The next brief and dashboard generation will pick it up automatically (follow-ups read from `TASKS.md` Active).

1. Read `~/cowork/TASKS.md`.
2. Insert a new line directly under the `## Active` header:
   ```
   - [ ] **<item>** - captured via /today on {YYYY-MM-DD}
   ```
   Preserve the rest of the file verbatim.
3. Print a one-line confirmation: "Added to TASKS.md Active: <item>"
4. Do NOT regenerate the dashboard or brief on `/today add`. The next bare `/today` or the 06:30 scheduled run will pick it up. (If the user wants it visible immediately, suggest they run bare `/today` next.)

## Argument parsing

- No args, or just `/today` → Mode 1.
- `/today add "<text>"` or `/today add <text>` → Mode 2 with `<text>` as the item.
- Anything else → print usage and stop.

## Voice

Match `personal/CLAUDE.md`. Active voice, short sentences, no filler. No em dashes. The brief leads with what changes the day.

## File references

- Spec: `spec/mission-control-spec.md`
- Workflow this mirrors: `personal-morning-brief` (scheduled, 06:30 PT daily)
- Domain voice: `personal/CLAUDE.md`
- Task list: `~/cowork/TASKS.md`
