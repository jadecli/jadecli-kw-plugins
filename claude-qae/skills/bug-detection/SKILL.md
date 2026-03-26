---
name: bug-detection
description: Patterns for finding bugs — anti-patterns from the playbook, edge cases, regression triggers, and common failure modes in agentic systems.
argument-hint: "<anti-patterns|edge-cases|regressions>"
---

# Bug Detection Patterns

## Playbook Anti-Patterns (catch these in review)

| Anti-Pattern | What to Look For | Fix |
|-------------|------------------|-----|
| Monolithic tool | Single MCP tool doing 3+ distinct operations | Split into granular single-purpose tools |
| Fragile enum expansion | Enum types growing with each edge case | Add `"other"` + `detail` field |
| Sequential file reading | Read tool loading all files one by one | Grep → pinpoint → Read specific |
| Context contamination | A/B exploration in single thread | Use `fork_session` for divergent branches |
| Emphatic prompt compliance | "CRITICAL POLICY: NEVER..." in system prompt | Application-layer hook intercept |
| Stale tool results | Resuming session with hours-old data | Filter out old `tool_result` on resume |
| Self-review bias | Same session reviews its own generated code | Independent review instance |
| Generic error strings | MCP returning "Operation failed" | Structured `{isError, errorCategory, isRetryable}` |

## Edge Cases to Test

### Agent Orchestration
- Coordinator receives empty result from subagent
- Subagent exceeds turn limit
- Parallel subagent spawning — race condition in result aggregation
- Tool call returns `isError: true` but `isRetryable: false`

### MCP Integration
- MCP server disconnects mid-tool-call
- Tool returns valid empty result (no matches) vs access failure
- Token expires between consecutive tool calls
- Tool description keyword conflicts with system prompt instructions

### Data Pipeline
- OTel event with null session_id
- Duplicate events (same event_id, different recorded_at)
- Token usage event with 0 cost (cache hit)
- Schema migration applied twice (idempotency)

### Plugin System
- Plugin installed at both user and project scope — which wins?
- Marketplace update overwrites local changes
- Skill invoked with no arguments when `argument-hint` expects one
- Two plugins define skills with same name

### Mobile/Integration
- Network drops during Remote Control session
- Dispatch task sent while Desktop is sleeping
- Channel message from non-allowlisted sender
- Scheduled task fires while Claude is mid-response

## Regression Triggers

Watch these areas after changes:
- Any change to `CLAUDE.md` → verify all plugins still load correctly
- Any change to `.mcp.json` → verify all MCP servers connect
- Any change to `hooks/` → verify pre/post tool behavior
- Any schema migration → verify Cube.js models still query correctly
- Any package version bump → verify `versions.json` in sync
