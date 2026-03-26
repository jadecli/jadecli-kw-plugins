---
name: scheduled-tasks
description: Create, list, update, and delete Claude Code scheduled tasks, remote triggers, and cron jobs. Manages recurring agent execution.
argument-hint: "<create|list|update|delete|run> [task-name]"
---

# /schedule - Manage Scheduled Tasks & Cron Jobs

## Usage

```
/schedule list
/schedule create "nightly ast rebuild" --cron "0 2 * * *" --prompt "/ast-index"
/schedule create "weekly audit" --cron "0 9 * * 1" --prompt "/packages audit"
/schedule update <id> --cron "0 3 * * *"
/schedule delete <id>
/schedule run <id>
```

## Common Schedules

| Name | Cron | Prompt |
|------|------|--------|
| Nightly AST rebuild | `0 2 * * *` | `/ast-index` |
| Weekly package audit | `0 9 * * 1` | `/packages audit` |
| Daily crawler run | `0 4 * * *` | `python crawler/run.py --all` |
| Monthly dep update | `0 10 1 * *` | `/packages update --all` |

## Implementation

Uses Claude Code's built-in scheduling:

```bash
# Create
claude schedule create \
  --name "nightly-ast-rebuild" \
  --cron "0 2 * * *" \
  --project . \
  --prompt "Run /ast-index to rebuild the AST index"

# List
claude schedule list

# Delete
claude schedule delete <trigger-id>
```

## Output

```
SCHEDULED TASKS
━━━━━━━━━━━━━━
ID      Name                 Cron          Last Run      Status
t-001   nightly ast rebuild  0 2 * * *     2026-03-26    ok
t-002   weekly pkg audit     0 9 * * 1     2026-03-24    ok
```

## Safety

- Scheduled tasks inherit project permissions from `.claude/settings.json`
- Destructive operations never auto-approved in scheduled runs
- Max 10 active schedules per project
