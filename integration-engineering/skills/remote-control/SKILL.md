---
name: remote-control
description: Drive a running local Claude Code session from claude.ai/code or the Claude mobile app. Continue work from any device while code runs on your machine.
argument-hint: "<start|connect|status|server>"
---

# Remote Control — Drive Sessions from Any Device

Continue a local Claude Code session from your phone, tablet, or any browser. Code runs locally; you steer it remotely.

## Quick Start

### Start a Remote Control server

```bash
claude remote-control
# Or with a name:
claude remote-control --name "jade-cowork-agents"
```

Press spacebar to show QR code for phone access.

### From an existing session

```
/remote-control
# Or:
/rc
```

### Start interactive session with remote access

```bash
claude --remote-control
# Or:
claude --rc
```

## Server Mode Flags

| Flag | Description |
|------|-------------|
| `--name "title"` | Custom session title visible in claude.ai/code |
| `--spawn same-dir` | All sessions share cwd (default) |
| `--spawn worktree` | Each session gets its own git worktree |
| `--capacity N` | Max concurrent sessions (default: 32) |
| `--verbose` | Detailed connection logs |
| `--sandbox` | Enable filesystem/network isolation |

## Connect from Another Device

1. **URL**: Open the session URL shown in terminal
2. **QR code**: Scan with phone camera → opens in Claude app
3. **claude.ai/code**: Find session in list (green dot = online)

## Enable for All Sessions

Run `/config` inside Claude Code → set "Enable Remote Control for all sessions" to `true`.

## Architecture

- Local machine makes outbound HTTPS only — no inbound ports
- All traffic through Anthropic API over TLS
- Session reconnects automatically after sleep/network drops
- Multiple short-lived credentials, each scoped and expiring independently

## Requirements

- Claude Pro, Max, Team, or Enterprise (API keys not supported)
- claude.ai OAuth login (not API key)
- Team/Enterprise: admin must enable Remote Control toggle
- Claude Code v2.1.51+

## When to Use

- Continue work from couch/phone after starting at desk
- Steer in-progress sessions without walking back to terminal
- Monitor long-running tasks from mobile
- Use both terminal and remote simultaneously — conversation stays in sync
