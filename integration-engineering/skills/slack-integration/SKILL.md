---
name: slack-integration
description: Mention @Claude in Slack channels to spawn Claude Code web sessions. Bug fixes, code reviews, and PRs directly from team chat.
argument-hint: "<setup|routing|troubleshoot>"
---

# Slack Integration — @Claude in Team Channels

Mention @Claude in Slack with a coding task → Claude Code session spawns on the web → PR comes back to Slack.

## Setup

1. **Install**: Workspace admin adds Claude from [Slack App Marketplace](https://slack.com/marketplace/A08SF47R6P4)
2. **Connect**: Claude App Home → Connect → authenticate with claude.ai
3. **GitHub**: Connect repos at claude.ai/code
4. **Routing**: Choose mode in App Home:
   - **Code only**: all @mentions → Claude Code sessions
   - **Code + Chat**: auto-routes between Code and Chat based on intent
5. **Invite**: `/invite @Claude` in channels where you want it

## How It Works

1. @mention Claude with a coding request in a channel/thread
2. Claude detects coding intent
3. Claude Code web session created automatically
4. Status updates posted to your Slack thread
5. Completion summary with action buttons:
   - **View Session** — full transcript on claude.ai/code
   - **Create PR** — open PR from session changes
   - **Retry as Code** — if routed to Chat, retry as Code
   - **Change Repo** — select different repository

## Context Gathering

- **In threads**: gathers all thread messages for context
- **In channels**: looks at recent channel messages
- Claude auto-selects repository based on conversation context

## Use Cases

```
@Claude fix the failing test in packages/auth/login.test.ts
```

```
@Claude investigate the null pointer in the error log above and open a PR with the fix
```

```
@Claude refactor the UserService class to use dependency injection
```

## Access Model

| Level | Control |
|-------|---------|
| User sessions | Run under your own Claude account |
| Rate limits | Count against your plan |
| Repo access | Only repos you've connected |
| Channel access | Claude must be invited to channel |

## Requirements

- Claude Pro, Max, Teams, or Enterprise
- Claude Code on the web enabled
- GitHub account connected with repos authenticated
- Slack workspace with Claude app installed

## Routing Modes

| Mode | Behavior |
|------|----------|
| Code only | All @mentions → Claude Code sessions |
| Code + Chat | Auto-routes: coding tasks → Code, general → Chat |

In Code + Chat mode, click "Retry as Code" if Claude mis-routes to Chat.

## Limitations

- GitHub only (no GitLab/Bitbucket)
- One PR per session
- Rate limits apply per user
- Channels only (no DMs)
- Web access required

## When to Use

- Bug reports in Slack → immediate fix
- Code review discussions → Claude implements feedback
- Team provides context → Claude uses it for debugging
- Async task delegation → work while Claude codes
