---
name: directives
description: Non-negotiable architectural rules governing the jadecli ecosystem. Violations fail the build. Derived from cert exam + playbook.
argument-hint: "<list|check|enforce>"
---

# Architectural Directives

Violations of any directive fail the build. No exceptions.

## Auth

- **D-AUTH-1**: All Claude auth uses `CLAUDE_CODE_OAUTH_TOKEN`. Never `ANTHROPIC_API_KEY`. Applies to workflows, env vars, secrets, SDK calls, cron jobs.
- **D-AUTH-2**: Secrets never in code. Environment variables via managed settings or `.env` (gitignored).

## Compliance

- **D-COMP-1**: Zero-tolerance policies enforced by application-layer hooks, never prompt instructions. Prompts have ~3% failure rate. Hooks have 0%.
- **D-COMP-2**: Every MCP tool returns structured errors (`isError`, `errorCategory`, `isRetryable`). Generic "Operation failed" strings are forbidden.
- **D-COMP-3**: Supabase tables have RLS enabled. No exceptions.

## Architecture

- **D-ARCH-1**: Model-driven tool selection. Claude reasons about which tool to call. No pre-configured decision trees unless compliance requires it.
- **D-ARCH-2**: Subagents have isolated context. They do not inherit parent conversation. All context must be explicitly passed.
- **D-ARCH-3**: Each subagent gets only the tools relevant to its role. 4-5 tools max. 18 tools degrades selection reliability.
- **D-ARCH-4**: Never default to real-time API for asynchronous needs. Batch API (50% savings) for overnight/weekly workloads.
- **D-ARCH-5**: Split monolithic tools into granular single-purpose tools. `analyze_dependencies` → `list_imports` + `resolve_transitive_deps` + `detect_circular_deps`.

## Context

- **D-CTX-1**: Never sequentially Read all files. Start broad (Grep for entry points), then pinpoint (Read specific functions).
- **D-CTX-2**: Tool results pruned before entering context. Extract only relevant fields, remove verbose data.
- **D-CTX-3**: Long sessions (>30 min) use scratchpad file pattern. Key findings, architectural maps, decisions in structured file — not raw message history.
- **D-CTX-4**: Session resumption: filter out stale `tool_result` messages. Force agent to re-fetch current data.

## Schema

- **D-SCHEMA-1**: Resilient catch-all enums. Add `"other"` + `detail` string field. Never fragile enum expansion.
- **D-SCHEMA-2**: Schema redundancy for validation. Extract `calculated_total` AND `stated_total`. Flag when `!=`.
- **D-SCHEMA-3**: Nullable fields for optional data. Never let model fabricate values to satisfy required fields.

## Data

- **D-DATA-1**: Supabase is the single data layer. OTel events, Cube.js models, agent telemetry — all in Supabase PostgreSQL.
- **D-DATA-2**: Netlify deploys jadecli.com. No other hosting.
- **D-DATA-3**: Linear for task management. Product roadmap, sprints, issues — all in Linear.
- **D-DATA-4**: 1 table per file. DDL + business logic version controlled, reviewed via PR. (Kimball methodology)

## Integration

- **D-INT-1**: iOS mobile must work with Slack, Cowork, and Dispatch. Not desktop-only.
- **D-INT-2**: All knowledge worker plugins must have functional MCP connections — not placeholder tokens.
- **D-INT-3**: Real data in all environments. No mock data past M1 milestone.
