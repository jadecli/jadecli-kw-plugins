---
name: claude-code-sdk-agents
description: Build custom agents with the Claude Agent SDK. Programmatic Claude Code via CLI (-p flag), Python, and TypeScript. Structured output, streaming, tool approval, and CI/CD scripting.
argument-hint: "<cli|python|typescript|examples>"
---

# Claude Agent SDK — Programmatic Claude Code

Run Claude Code programmatically from CLI, Python, or TypeScript. Same tools, agent loop, and context management as interactive mode.

## CLI Usage (`claude -p`)

### Basic

```bash
claude -p "Find and fix the bug in auth.py" --allowedTools "Read,Edit,Bash"
```

### Bare Mode (CI/scripts — skip all auto-discovery)

```bash
claude --bare -p "Summarize this file" --allowedTools "Read"
```

Skips hooks, skills, plugins, MCP servers, CLAUDE.md. Only explicit flags take effect.

### Structured JSON Output

```bash
claude -p "Summarize this project" --output-format json
```

### JSON Schema Output

```bash
claude -p "Extract function names from auth.py" \
  --output-format json \
  --json-schema '{"type":"object","properties":{"functions":{"type":"array","items":{"type":"string"}}},"required":["functions"]}'
```

### Streaming

```bash
claude -p "Explain recursion" --output-format stream-json --verbose --include-partial-messages
```

Filter text deltas:
```bash
claude -p "Write a poem" --output-format stream-json --verbose --include-partial-messages | \
  jq -rj 'select(.type == "stream_event" and .event.delta.type? == "text_delta") | .event.delta.text'
```

### Auto-Approve Tools

```bash
claude -p "Run tests and fix failures" --allowedTools "Bash,Read,Edit"
```

### Create a Commit

```bash
claude -p "Look at staged changes and create a commit" \
  --allowedTools "Bash(git diff *),Bash(git log *),Bash(git status *),Bash(git commit *)"
```

### Custom System Prompt

```bash
gh pr diff "$1" | claude -p \
  --append-system-prompt "You are a security engineer. Review for vulnerabilities." \
  --output-format json
```

### Continue Conversations

```bash
claude -p "Review this codebase for performance issues"
claude -p "Now focus on database queries" --continue
claude -p "Generate summary of all issues" --continue
```

### Resume Specific Session

```bash
session_id=$(claude -p "Start review" --output-format json | jq -r '.session_id')
claude -p "Continue that review" --resume "$session_id"
```

## Bare Mode Flags

| To load | Use |
|---------|-----|
| System prompt | `--append-system-prompt`, `--append-system-prompt-file` |
| Settings | `--settings <file-or-json>` |
| MCP servers | `--mcp-config <file-or-json>` |
| Custom agents | `--agents <json>` |
| Plugin dir | `--plugin-dir <path>` |

## Python & TypeScript SDKs

Full programmatic control with callbacks, message objects, and tool approval:

```bash
# Python
pip install claude-agent-sdk

# TypeScript
npm install @anthropic-ai/claude-agent-sdk
```

See https://platform.claude.com/docs/en/agent-sdk/overview

## Stream Events

| Field | Type | Description |
|-------|------|-------------|
| `type` | `"system"` | Message type |
| `subtype` | `"api_retry"` | Retry event |
| `attempt` | integer | Current attempt |
| `error` | string | Error category |

## CI/CD Patterns

### GitHub Actions

```yaml
- run: |
    claude --bare -p "Review PR diff for security issues" \
      --allowedTools "Read,Bash(git diff *)" \
      --output-format json > review.json
```

### Pipe Input

```bash
cat error.log | claude -p "Diagnose this error and suggest a fix"
```

### Multi-Step Pipeline

```bash
claude --bare -p "Run tests" --allowedTools "Bash" --output-format json | \
  jq -r '.result' | \
  claude --bare -p "Fix the failures described above" --allowedTools "Read,Edit"
```

## Requirements

- Claude Code v2.1.72+ for scheduled tasks
- `CLAUDE_CODE_OAUTH_TOKEN` (Pro/Max) or `ANTHROPIC_API_KEY` (API users) for bare mode
- `--bare` recommended for all scripted/CI usage
