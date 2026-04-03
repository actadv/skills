---
name: loopit
description: Start a continuous development loop — runs /develop, /pr-review, and /e2e on a recurring schedule and /bob-review once daily
user-invocable: true
argument-hint: "[start|stop|status] [slow|normal|fast|turbo]"
allowed-tools: CronCreate, CronDelete, CronList, Bash, Skill
---

You orchestrate recurring development and review schedules.

## Speed tiers

Map the speed argument to cron schedules:

| Speed | Core tasks | Bob-review | Use case |
|---|---|---|---|
| `slow` | Every 30 min (`3,33 * * * *`) | Daily 9:17 AM | Background monitoring |
| `normal` (default) | Every 15 min (`10,25,40,55 * * * *`) | Daily 9:17 AM | Active development |
| `fast` | Every 5 min (`*/5 * * * *`) | Daily 9:17 AM | Sprint mode |
| `turbo` | Every 2 min (`*/2 * * * *`) | Every 6 hours (`17 */6 * * *`) | Crunch time |

## Commands

### stop

`CronList`, then `CronDelete` every job whose prompt contains `/develop`, `/pr-review`, `/e2e`, or `/bob-review`. Confirm what was stopped.

### status

`CronList`. Show a table of active loops (schedule, prompt, speed). If none, say so.

Also show the global agent count:

```bash
bash ~/.claude/agent-registry.sh count
```

And the limits:

```bash
cat ~/.claude/agent-limits.json
```

### start (default)

Parse `$ARGUMENTS` for a speed tier. If no speed given, default to `normal`. If only `start` is given, default to `normal`.

First `CronList` to check for duplicates — skip creation if already running.

Then create four schedules using the cron expressions for the chosen speed:

```
CronCreate  cron: "<core-cron>"   prompt: "/develop"     recurring: true
CronCreate  cron: "<core-cron>"   prompt: "/pr-review"   recurring: true
CronCreate  cron: "<core-cron>"   prompt: "/e2e"         recurring: true
CronCreate  cron: "<bob-cron>"    prompt: "/bob-review"  recurring: true
```

Confirm what speed was set and show the schedule.
