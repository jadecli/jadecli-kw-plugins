---
name: primitives
description: Fundamental building blocks available to all knowledge workers. Tools, protocols, patterns, and infrastructure components.
argument-hint: "<list|describe|check> [primitive-name]"
---

# Primitives — Building Blocks

## Tooling Primitives

| Primitive | Type | Purpose |
|-----------|------|---------|
| Claude Agent SDK | SDK | Programmatic Claude Code (Python, TypeScript, CLI `-p`) |
| MCP (Model Context Protocol) | Protocol | Tool and resource interfaces for backend integration |
| Hooks (Pre/PostToolUse) | Enforcement | Intercept tool calls for compliance, normalization, gating |
| Skills (SKILL.md) | Knowledge | On-demand domain knowledge loaded by model when relevant |
| Commands (.claude/commands/) | Workflow | User-invoked slash commands for structured workflows |
| Agents (.claude/agents/) | Role | Specialized agent definitions with model, tools, system prompt |
| Plugins (.claude-plugin/) | Distribution | Shareable packages of skills, agents, hooks, MCP configs |
| CLAUDE.md | Memory | Project instructions loaded at session start |
| .claude/rules/ | Standards | Path-scoped rules loaded conditionally via glob patterns |

## Infrastructure Primitives

| Primitive | Service | Role in jadecli |
|-----------|---------|----------------|
| Supabase | PostgreSQL + Auth + Edge Functions | Data layer — OTel events, Cube.js models, app data |
| Netlify | CDN + Edge + Functions | Hosts jadecli.com, edge functions, forms |
| Linear | Project management | Task tracking, sprints, product roadmap |
| GitHub Actions | CI/CD | Automated review, test, deploy pipelines |
| Slack | Team communication | @Claude mentions → Claude Code web sessions |
| OTel Collector | Telemetry | Metrics + events → Supabase |
| Cube.js | Semantic layer | Pre-aggregations, measures, dimensions on Supabase |
| Tree-sitter | AST parsing | L1-L4 codebase index for semantic search |

## Communication Primitives

| Primitive | Trigger | Runs on |
|-----------|---------|---------|
| Dispatch | Mobile app message | Your machine (Desktop) |
| Remote Control | claude.ai/code or mobile | Your machine (CLI/VS Code) |
| Channels | Telegram, Discord, iMessage, webhook | Your machine (CLI) |
| Slack | @Claude in channel | Anthropic cloud |
| Scheduled Tasks | Cron / /loop | CLI, Desktop, or cloud |

## Architecture Patterns (from Playbook)

| Pattern | Anti-Pattern | Fix |
|---------|-------------|-----|
| Granular tools | Monolithic tool | Split into single-purpose tools with clear descriptions |
| Resilient catch-all schemas | Fragile enum expansion | Add `"other"` + `detail` field |
| Directed exploration | Sequential file reading | Grep → pinpoint → Read |
| Scratchpad file | Raw message history bloat | Structured findings file for long sessions |
| fork_session branching | Mixed context A/B | Separate branches for divergent exploration |
| Tool context pruning | Verbose tool results | Application-side filter before context |
| Zero-tolerance hooks | Emphatic prompt compliance | PreToolUse hooks block policy violations |
| Schema redundancy | Trust extracted totals | calculated_total + stated_total, flag mismatch |
| Session compression | Full verbatim history | Summarize resolved → verbatim only for active issue |

## 3-Layer Architecture (from Playbook)

```
Layer 3: Validation, Aggregation & Delivery
  ├── PostProcessing, ResponseComposer
  ├── Schema validation, dedup, ranking
  └── Final output to user/system

Layer 2: Multi-Agent Orchestration & Generation
  ├── Agent CodeGenerator, Agent DataAnalyzer
  ├── OrchestrationScheduler, Agent Communication
  └── Coordinator → subagent delegation → synthesis

Layer 1: Ingestion & Routing
  ├── QueryRouter, IntentClassifier
  ├── InputHandling, SafetyCheck
  └── Route to appropriate agent/tool pipeline
```

## Constraint Hierarchy (from Playbook)

```
Latency    → mitigated by parallelization & caching
Accuracy   → mitigated by structured intermediates & few-shot prompts
Cost       → mitigated by batch APIs & context pruning
Compliance → enforced by application-layer intercepts (NOT prompts)
```
