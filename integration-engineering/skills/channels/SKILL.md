---
name: channels
description: Push events from Telegram, Discord, iMessage, or custom webhooks into a running Claude Code session. Two-way chat bridges and CI event receivers.
argument-hint: "<setup|telegram|discord|imessage|webhook>"
---

# Channels — Push Events into Running Sessions

Channels are MCP servers that push events into your Claude Code session. Claude reacts while you're away. Two-way: Claude reads events and replies through the same channel.

## Supported Channels

### Telegram

```bash
# Install
/plugin install telegram@claude-plugins-official

# Configure
/telegram:configure <BOT_TOKEN>

# Start with channel
claude --channels plugin:telegram@claude-plugins-official

# Pair (send any message to your bot, get code)
/telegram:access pair <code>
/telegram:access policy allowlist
```

### Discord

```bash
# Install
/plugin install discord@claude-plugins-official

# Configure
/discord:configure <BOT_TOKEN>

# Start with channel
claude --channels plugin:discord@claude-plugins-official

# Pair
/discord:access pair <code>
/discord:access policy allowlist
```

Discord bot needs: View Channels, Send Messages, Send Messages in Threads, Read Message History, Attach Files, Add Reactions. Enable Message Content Intent in developer portal.

### iMessage (macOS only)

```bash
# Install
/plugin install imessage@claude-plugins-official

# Start with channel
claude --channels plugin:imessage@claude-plugins-official

# Text yourself — works immediately
# Allow others:
/imessage:access allow +15551234567
```

Requires Full Disk Access for Messages database. Grant in System Settings → Privacy & Security → Full Disk Access.

### Fakechat (demo/testing)

```bash
/plugin install fakechat@claude-plugins-official
claude --channels plugin:fakechat@claude-plugins-official
# Open http://localhost:8787
```

## Multiple Channels

```bash
claude --channels plugin:telegram@claude-plugins-official plugin:discord@claude-plugins-official
```

## Custom Webhook Receiver

Build your own channel to receive CI failures, deploy events, or monitoring alerts. See https://code.claude.com/docs/en/channels-reference

## Security

- Every channel maintains a sender allowlist
- Only paired/approved IDs can push messages
- Being in .mcp.json isn't enough — server must be named in `--channels`
- Permission relay: approved senders can approve/deny tool use remotely

## Enterprise Controls

| Setting | Purpose |
|---------|---------|
| `channelsEnabled` | Master switch (default: off for Team/Enterprise) |
| `allowedChannelPlugins` | Restrict which plugins can register |

## Requirements

- Claude Code v2.1.80+
- claude.ai login (not API key)
- Bun runtime installed
- Research preview — syntax may change

## When to Use

- Chat bridge: ask Claude from phone via Telegram/Discord/iMessage
- CI webhook: push build failures directly into active session
- Monitoring: alert Claude when error rate spikes
- React to events as they happen (vs polling with /loop)
