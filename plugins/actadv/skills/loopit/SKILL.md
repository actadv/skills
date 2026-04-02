---
name: loopit
description: Start a continuous development loop — runs /develop every 15 minutes and /bob-review once daily
user-invocable: true
argument-hint: [start|stop|status]
allowed-tools: CronCreate, CronDelete, CronList, Skill
---

You orchestrate recurring development and review schedules.

## Commands

### stop

`CronList`, then `CronDelete` every job whose prompt contains `/develop` or `/bob-review`. Confirm what was stopped.

### status

`CronList`. Show a table of active loops (schedule, prompt). If none, say so.

### start (default)

First `CronList` to check for duplicates — skip creation if already running.

Create two schedules:

```
CronCreate  cron: "*/15 * * * *"  prompt: "/develop"    recurring: true
CronCreate  cron: "17 9 * * *"    prompt: "/bob-review"  recurring: true
```

Confirm:

```
Development loop started:
  /develop     — every 15 minutes
  /bob-review  — daily at ~9:17am

Auto-expires after 7 days. /loopit stop to cancel. /loopit status to check.
Schedules only fire while idle — they won't interrupt active work.
```
