---
name: dispatch
description: Send coding tasks from the Claude mobile app to a Desktop session on your machine. Pair your phone with Claude Desktop for delegating work while away.
argument-hint: "<setup|send|status>"
---

# Dispatch — Send Tasks from Your Phone

Dispatch lets you message a task from the Claude mobile app and it spawns a Desktop session on your machine.

## How It Works

1. **Trigger**: Message a task from the Claude mobile app
2. **Runs on**: Your machine via Claude Desktop
3. **Setup**: Pair the mobile app with Desktop

## Setup

1. Install Claude Desktop on your machine
2. Install the Claude app on iOS or Android
3. Pair: open Claude mobile → Settings → Pair with Desktop
4. Follow the pairing flow (see https://support.claude.com/en/articles/13947068)

## Usage

From your phone, open the Claude app and send a coding task:

```
Fix the failing test in packages/auth/login.test.ts
```

Claude Desktop picks it up, runs it against your local files, and you can check progress from your phone.

## Requirements

- Claude Pro or Max subscription
- Claude Desktop installed and running
- Claude mobile app paired with Desktop
- Machine must be awake and online

## When to Use

- Delegating work while commuting or away from desk
- Quick fixes you think of while on your phone
- Minimal setup — just pair once and go
