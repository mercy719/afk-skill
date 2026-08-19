# AFK Work Loop

An AFK skill for Codex that scans project context, reconciles unfinished work, proposes a reviewable execution plan, and waits for approval before starting a persistent goal.

![AFK Work Loop](assets/afk-work-loop.png)

## Install

Copy this repository into the Codex skills directory as `afk`:

```text
~/.codex/skills/afk/
```

## Use

Invoke `$afk`, or use one of these modes:

- `afk today` — discover unfinished work from today's project sessions
- `afk resume` — continue the active goal
- `afk report` — report active goal status without starting new work
- `afk <objective>` — propose a plan for a concrete objective
