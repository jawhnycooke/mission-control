# mission-control

A Claude Code / Cowork plugin with two on-demand Mission Control skills for a personal Cowork workspace.

## Skills

- `mission-control:today` — re-run the morning brief and regenerate the dashboard, or append a follow-up to `TASKS.md` with `/today add "<item>"`.
- `mission-control:prep` — print a one-screen pre-read for an upcoming meeting: attendees, relationship context, open threads, and what to bring.

Skills are namespaced by the plugin, so they run as `/mission-control:today` and `/mission-control:prep`.

## Install

In Claude Code:

```
/plugin marketplace add jawhnycooke/mission-control
/plugin install mission-control@mission-control
```

For local testing without installing, point Claude Code at a clone of this repo:

```
claude --plugin-dir /path/to/mission-control
```

## Requirements

These skills read and write files in a Cowork "Mission Control" workspace under `~/cowork/` (for example `TASKS.md`, `personal/data/*.json`, and `spec/mission-control-spec.md`) and assume that workspace exists. They are read-only against connectors and never send email, Slack messages, or calendar invites, and never write to `personal/inputs/`.

## Structure

```
.claude-plugin/
├── marketplace.json   # single-plugin marketplace (source: "./")
└── plugin.json        # plugin manifest
skills/
├── today/SKILL.md
└── prep/SKILL.md
```
