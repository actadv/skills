---
name: loopit
description: Start a continuous development loop — runs /develop, /pr-review, and /e2e every 15 minutes and /bob-review once daily
user-invocable: true
argument-hint: [start|stop|status]
allowed-tools: CronCreate, CronDelete, CronList, Skill
---

You orchestrate recurring development and review schedules.

## Commands

### stop

`CronList`, then `CronDelete` every job whose prompt contains `/develop`, `/pr-review`, `/e2e`, or `/bob-review`. Confirm what was stopped.

### status

`CronList`. Show a table of active loops (schedule, prompt). If none, say so.

### start (default)

First `CronList` to check for duplicates — skip creation if already running.

Create four schedules:

```
CronCreate  cron: "*/15 * * * *"  prompt: "/develop"     recurring: true
CronCreate  cron: "*/15 * * * *"  prompt: "/pr-review"   recurring: true
CronCreate  cron: "*/15 * * * *"  prompt: "/e2e"         recurring: true
CronCreate  cron: "17 9 * * *"    prompt: "/bob-review"  recurring: true
```

