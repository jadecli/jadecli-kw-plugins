---
name: d1-agentic-orchestration
description: "Cert Domain 1 (27%): Agentic loops, multi-agent coordination, subagent spawning, handoff patterns, hook enforcement, task decomposition, session state."
argument-hint: "<task-statement-number>"
---

# D1: Agentic Architecture & Orchestration (27%)

## 1.1 Agentic Loop Control
- Loop continues when `stop_reason` = `tool_use`, terminates on `end_turn`
- Tool results appended for next iteration reasoning
- Anti-patterns: NL parsing for termination, arbitrary caps, text-content completion checks

## 1.2 Coordinator-Subagent Patterns
- Hub-and-spoke: coordinator manages all inter-agent communication
- Subagents: isolated context, no inherited history
- Coordinator: decompose → delegate → aggregate → refine if gaps
- Partition scope across subagents to minimize duplication

## 1.3 Subagent Spawning
- `Task` tool spawns; `allowedTools` must include `"Task"`
- Context explicitly provided — subagents don't inherit parent
- `AgentDefinition`: description, system prompt, tool restrictions
- `fork_session` for A/B divergent exploration
- Parallel: emit multiple `Task` calls in one coordinator turn

## 1.4 Enforcement & Handoff
- Hooks for deterministic compliance; prompts for probabilistic guidance
- Programmatic prerequisites: block downstream until upstream completes
- Structured handoff: `{customer_id, root_cause, amount, recommended_action}`

## 1.5 Hook Interception
- `PostToolUse`: normalize data (timestamps, status codes) before agent processes
- `PreToolUse`: block policy violations, redirect to escalation
- Hooks > prompts when business rules require guaranteed compliance

## 1.6 Task Decomposition
- Fixed sequential (prompt chaining) for predictable multi-aspect reviews
- Dynamic adaptive for open-ended investigation
- Large reviews: per-file local passes + cross-file integration pass

## 1.7 Session Management
- `--resume <name>` for named continuation
- `fork_session` for parallel branches
- Fresh + structured summary > resume with stale tool results
- Inform resumed session about specific file changes

## jadecli Mapping
- Staff review (6 agents): 1.2 coordinator pattern
- Skeptical codegen (6 agents): 1.2 adversarial variant
- route-model hook: 1.5 pre-tool routing
- plan-gate hook: 1.4 enforcement
- test-runner hook: 1.5 post-tool validation
