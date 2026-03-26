---
name: plugin-inventory
description: Complete inventory of all activated knowledge worker plugins, MCPs, skills, and integrations in the jadecli ecosystem.
argument-hint: "<list|check|status>"
---

# Plugin Inventory — All Activated Components

## Knowledge Worker Plugins (8)

| Plugin | Skills | Status |
|--------|--------|--------|
| data-engineering | package-manager, ast-index, scheduled-tasks, supabase-cube | M3 |
| frontend-engineering | 12 Netlify skills (deploy, edge, functions, frameworks) | M3 |
| integration-engineering | 9 skills (dispatch, remote-control, channels, slack, actions, sdk, security-review, plugins) | M3 |
| analytics-engineering | 6 skills (telemetry, otel-queries, cost, tools, sessions, kimball) | M3 |
| software-dev-manager | 6 skills (otel-setup, usage, cost-mgmt, org-analytics, kw-metrics, rate-limits) | M3 |
| claude-architect | directives, primitives, 5 domain skills, plugin-inventory, milestones | M3 |
| claude-qae | directives, primitives, test-strategy, code-standards, bug-detection, ci-cd-gates, e2e-validation, milestone-tests | M3 |
| engineering (anthropic) | 10 skills (architecture, code-review, debug, deploy, docs, incident, standup, system-design, tech-debt, testing) | M3 |

## MCP Servers

| Server | Purpose | Milestone |
|--------|---------|-----------|
| @supabase/mcp-server-supabase | Data layer — OTel, Cube.js, app data | M2 |
| @modelcontextprotocol/server-github | PRs, issues, repos, code search | M2 |
| jade-console-mcp | jadecli.com dashboard queries | M2 |
| Linear MCP | Task management, roadmap, sprints | M2 |
| Cube.js REST API | Semantic layer queries | M7 |

## Integrations

| Integration | Type | Milestone |
|-------------|------|-----------|
| GitHub Actions (claude-code-action@v1) | CI/CD | M6 |
| Code Review (managed, multi-agent) | CI/CD | M6 |
| Slack (@Claude in channels) | Communication | M5 |
| Channels (Telegram, Discord, iMessage) | Communication | M5 |
| Remote Control (claude.ai/code, mobile) | Remote access | M5 |
| Dispatch (mobile → Desktop) | Remote access | M5 |
| Scheduled Tasks (cloud, Desktop, /loop) | Automation | M5 |
| Netlify (jadecli.com) | Hosting | M8 |
| OTel Collector → Supabase | Observability | M1 |
| Tree-sitter AST index | Code intelligence | M1 |

## Agent Teams

| Team | Agents | Pattern |
|------|--------|---------|
| Staff Review | 6 specialists + lead orchestrator | D1.2 coordinator-subagent |
| Skeptical Codegen | 6 adversarial reviewers | D1.2 independent instances |
| 18 Agent Perspectives | abramov, carmack, hickey, etc. | D1.3 scoped subagents |

## Commands (16)

/plan, /commit, /verify, /code-review, /tdd, /eval, /threat-model, /grain, /lno, /ticket, /retro, /runbook, /commit-push-pr, /next-pr, /stack-prs, /update-claude-md
