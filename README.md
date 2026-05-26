# mission-control

A Claude Code / Cowork plugin with three on-demand Mission Control skills for a personal Cowork workspace.

## Skills

- `mission-control:today` — re-run the morning brief and regenerate the dashboard, or append a follow-up to `TASKS.md` with `/today add "<item>"`.
- `mission-control:prep` — print a one-screen pre-read for an upcoming meeting: attendees, relationship context, open threads, and what to bring.
- `mission-control:research` — deep-dive a stock or ETF with `/research <TICKER>`: pulls fundamentals and recent news from the Bigdata.com MCP, anchors the read to your investing philosophy, and writes a thesis note to `investments/outputs/research/`. Informs, never recommends buy/sell.

Skills are namespaced by the plugin, so they run as `/mission-control:today`, `/mission-control:prep`, and `/mission-control:research`.

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

`mission-control:research` additionally requires the Bigdata.com MCP for fundamentals and news, and reads `investments/inputs/holdings.csv`, `investments/data/holdings.json`, and `memory/investments/philosophy.md`. It writes only to `investments/outputs/research/`.

## Structure

```
.claude-plugin/
├── marketplace.json   # single-plugin marketplace (source: "./")
└── plugin.json        # plugin manifest
skills/
├── today/SKILL.md
├── prep/SKILL.md
└── research/SKILL.md
```
