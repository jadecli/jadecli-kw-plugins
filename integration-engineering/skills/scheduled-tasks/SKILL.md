---
name: scheduled-tasks
description: Run prompts on a schedule with /loop, CronCreate, Desktop tasks, and cloud tasks. Poll deployments, babysit PRs, run nightly reviews.
argument-hint: "<loop|create|list|cloud|desktop>"
---

# Scheduled Tasks — Run Prompts on a Schedule

Three scheduling tiers: session-scoped (/loop), Desktop persistent, and cloud persistent.

## Compare Options

| | Cloud | Desktop | /loop |
|---|---|---|---|
| Runs on | Anthropic cloud | Your machine | Your machine |
| Requires machine on | No | Yes | Yes |
| Requires open session | No | No | Yes |
| Persistent across restarts | Yes | Yes | No |
| Access to local files | No (fresh clone) | Yes | Yes |
| Min interval | 1 hour | 1 minute | 1 minute |

## /loop — Session-Scoped Scheduling

Quick scheduling within a running session:

```
/loop 5m check if the deployment finished and tell me what happened
/loop 20m /review-pr 1234
/loop check the build          # defaults to every 10 minutes
```

### Interval Syntax

| Form | Example | Parsed |
|------|---------|--------|
| Leading | `/loop 30m check build` | Every 30 min |
| Trailing | `/loop check build every 2h` | Every 2 hours |
| No interval | `/loop check build` | Every 10 min (default) |

Units: `s` (seconds), `m` (minutes), `h` (hours), `d` (days).

### One-Time Reminders

```
remind me at 3pm to push the release branch
in 45 minutes, check whether integration tests passed
```

## CronCreate — Underlying Tools

| Tool | Purpose |
|------|---------|
| `CronCreate` | Schedule new task (5-field cron, prompt, recur/once) |
| `CronList` | List all tasks with IDs, schedules, prompts |
| `CronDelete` | Cancel by 8-char ID |

Max 50 tasks per session. 3-day auto-expiry on recurring tasks.

### Cron Reference

| Expression | Meaning |
|------------|---------|
| `*/5 * * * *` | Every 5 minutes |
| `0 * * * *` | Every hour |
| `0 9 * * *` | Daily at 9am local |
| `0 9 * * 1-5` | Weekdays at 9am |
| `30 14 15 3 *` | March 15 at 2:30pm |

All times in local timezone. Day-of-week: 0/7=Sun through 6=Sat.

## Cloud Scheduled Tasks

Persistent, survives restarts, runs on Anthropic infrastructure:

```bash
# Via CLI
claude schedule create \
  --name "nightly-review" \
  --cron "0 2 * * *" \
  --prompt "Review open PRs and summarize"
```

Configure via `/schedule` in CLI or claude.ai/code web interface.

## Desktop Scheduled Tasks

Persistent, runs on your machine with local file access:

Configure in Claude Desktop → Settings → Scheduled Tasks. Supports MCP servers and connectors. Min interval: 1 minute.

## How Tasks Run

- Scheduler checks every second, enqueues at low priority
- Fires between your turns (not mid-response)
- If Claude is busy, waits until current turn ends
- Jitter: recurring tasks fire up to 10% of period late (max 15 min)
- One-shot tasks at :00/:30 fire up to 90s early

## Common Schedules

| Task | Cron | Prompt |
|------|------|--------|
| Nightly PR review | `0 2 * * *` | Review all open PRs |
| Deploy check | `*/5 * * * *` | Check deploy status |
| Weekly audit | `0 9 * * 1` | /packages audit |
| Daily AST rebuild | `0 3 * * *` | /ast-index |

## Disable

```bash
export CLAUDE_CODE_DISABLE_CRON=1
```

## Limitations

- /loop: session-scoped, lost on exit, no catch-up for missed fires
- Cloud: no local file access (fresh clone), 1-hour minimum
- Desktop: machine must be on

## When to Use

- **Cloud**: reliable unattended automation (nightly reviews, weekly reports)
- **Desktop**: local file access + persistence (build monitoring, test runs)
- **/loop**: quick polling during active work (deploy checks, PR babysitting)
