---
name: milestones
description: 10 build-passing checkpoints. If all milestones pass, the jadecli knowledge worker ecosystem is fully integrated end-to-end with real data, mobile, Slack, Cowork, Supabase, Netlify, and Linear.
argument-hint: "<M1|M2|...|M10|all|status>"
---

# Milestones — 10 Build Checkpoints

All milestones must pass. If any fails, the system is not production-ready.

---

## M1: Foundation

**Validates**: OTel pipeline, CLAUDE.md hierarchy, auth policy, AST index.

| Condition | Test |
|-----------|------|
| OTel events reach Supabase | `SELECT COUNT(*) FROM otel_events WHERE recorded_at > now() - INTERVAL '1 hour'` returns > 0 |
| CLAUDE.md loads at session start | `/memory` shows project CLAUDE.md |
| Auth policy enforced | `grep -r ANTHROPIC_API_KEY .github/` returns 0 matches |
| All secrets use CLAUDE_CODE_OAUTH_TOKEN | `grep -r CLAUDE_CODE_OAUTH_TOKEN .github/workflows/` matches all workflow auth |
| AST index builds | `python scripts/build-ast-index.py` exits 0 with L1-L4 populated |
| .gitignore correct | `.venv/`, `ast-index/`, `node_modules/` all excluded |

---

## M2: Tools & MCP

**Validates**: All MCP servers connected and responding with structured errors.

| Condition | Test |
|-----------|------|
| Supabase MCP responds | MCP tool call to `supabase:list_tables` returns table list |
| GitHub MCP responds | MCP tool call to `github:get_repo` returns repo metadata |
| Linear MCP responds | MCP tool call to `linear:list_issues` returns issues |
| jade-console-mcp responds | MCP tool call returns dashboard data |
| All MCP errors structured | Every MCP server returns `{isError, errorCategory, isRetryable}` on failure |
| Tool descriptions non-overlapping | No two tools share >80% description similarity |
| Tool count per agent ≤ 8 | Each subagent definition has ≤ 8 allowedTools |

---

## M3: Plugins

**Validates**: All 8 KW plugins installed, skills invocable, marketplace synced.

| Condition | Test |
|-----------|------|
| jadecli-kw-plugins marketplace added | `claude plugin marketplace list` shows jadecli-kw-plugins |
| All 8 plugins installed | `claude plugin list` shows 8 enabled plugins |
| Each plugin has ≥ 1 skill | Every plugin dir contains `skills/*/SKILL.md` |
| Skills invoke without error | `/plugin-name:skill-name` returns valid output for each |
| Marketplace manifest valid | `.claude-plugin/marketplace.json` passes JSON schema validation |
| Plugin versions set | Every `plugin.json` has semver version field |

---

## M4: Agents

**Validates**: Staff review team, skeptical codegen team, agent factory operational.

| Condition | Test |
|-----------|------|
| Staff review team passes | `cd staff-review-team && npm test` exits 0 |
| Skeptical codegen team passes | `cd skeptical-codegen-team && npm test` exits 0 |
| Agent factory spawns | `packages/agent-factory/` TypeScript compiles |
| 18 agent perspectives load | `.claude/agents/` contains 18 `.md` files |
| Router agent classifies correctly | router.md routes HAIKU/SONNET/OPUS for test prompts |
| Subagents have isolated context | No subagent definition inherits parent allowedTools |

---

## M5: Integrations

**Validates**: Slack, Channels, Remote Control, Dispatch — all functional.

| Condition | Test |
|-----------|------|
| Slack @Claude responds | @Claude mention in test channel creates Claude Code web session |
| Telegram channel paired | `/telegram:access` shows allowlisted sender |
| Discord channel paired | `/discord:access` shows allowlisted sender |
| iMessage channel works | Self-text → Claude reply in Messages |
| Remote Control connects | `claude remote-control` → session appears in claude.ai/code |
| Dispatch paired | Mobile app → Desktop session spawns |
| Scheduled task fires | `/loop 1m echo test` fires and produces output |
| Permission relay works | Channel can approve/deny tool use remotely |

---

## M6: CI/CD

**Validates**: GitHub Actions green, code review posting, security review passing.

| Condition | Test |
|-----------|------|
| claude.yml triggers on @claude | PR comment with @claude → workflow runs |
| claude-code-review.yml posts findings | PR open → review comments posted within 20 min |
| CI pipeline green on main | `gh run list --branch main --limit 1` shows success |
| Auth uses claude_code_oauth_token | All workflows use `claude_code_oauth_token` secret |
| Branch protection enforced | `gh api repos/jadecli/jadebot/branches/main/protection` returns rules |
| Squash merges only | Repo settings: merge commit disabled, squash enabled |
| Conventional commits enforced | commitlint hook rejects non-conventional title |

---

## M7: Data

**Validates**: Supabase tables populated with real OTel events, Cube.js queryable.

| Condition | Test |
|-----------|------|
| otel_events has real sessions | `SELECT COUNT(*) FROM otel_events WHERE event_name = 'claude_code.session.stop'` > 10 |
| otel_events has tool_results | `SELECT COUNT(*) FROM otel_events WHERE event_name = 'claude_code.tool_result'` > 100 |
| otel_events has token_usage | `SELECT COUNT(*) FROM otel_events WHERE event_name = 'claude_code.token_usage'` > 10 |
| RLS enabled on all tables | `SELECT tablename FROM pg_tables WHERE schemaname = 'public'` → all have RLS |
| Cube.js models defined | `cube/model/` contains ≥ 4 `.yml` files |
| Cube.js pre-aggregations refresh | `daily_by_role` pre-agg has data within 1 hour |
| Kimball DDL: 1 table per file | `schema/facts/` and `schema/dimensions/` each have individual `.sql` files |
| Pipeline metadata headers | Every `.sql` file starts with `-- DESCRIPTION:`, `-- GRAIN:`, `-- LOAD_TYPE:` |

---

## M8: Frontend

**Validates**: Netlify deploying jadecli.com, telemetry dashboard rendering real data.

| Condition | Test |
|-----------|------|
| jadecli.com resolves | `curl -s -o /dev/null -w '%{http_code}' https://jadecli.com` returns 200 |
| Netlify deploy succeeds | `netlify status` shows published deploy |
| Telemetry dashboard renders | `/telemetry` page returns HTML with StatCard components |
| Dashboard shows real data | TelemetrySummary.totalSessions > 0 (not mock) |
| Tool breakdown populated | ToolBreakdown array length > 0 |
| Cost cards show USD values | CostSummary.totalCostUsd > 0 |
| Vercel console deploys | apps/console build succeeds on Vercel |

---

## M9: Mobile

**Validates**: iOS Cowork + Slack + Dispatch end-to-end.

| Condition | Test |
|-----------|------|
| Cowork plugin functions on iOS | Cowork app invokes KW plugin skill successfully |
| Slack → Code session from phone | @Claude in Slack → session created, viewable in Claude app |
| Dispatch sends task from phone | Mobile app message → Desktop session spawns and completes |
| Remote Control from mobile | claude.ai/code mobile → drive local session |
| Permission relay from mobile | Mobile approves tool use during channel session |
| Notifications reach mobile | Claude Code task completion notification appears on phone |
| Session history syncs | Sessions created from mobile visible in claude.ai/code |

---

## M10: Roadmap

**Validates**: Linear integration, product roadmap tracked, WBR generating.

| Condition | Test |
|-----------|------|
| Linear MCP connected | `linear:list_issues` returns jadecli project issues |
| Product roadmap in Linear | Linear project has milestones with due dates |
| Sprint issues tracked | Current sprint has ≥ 5 issues with status |
| WBR cron fires | `wbr-cron.yml` ran successfully this week |
| WBR output stored | WBR report generated with real metrics (not placeholder) |
| Knowledge worker ROI tracked | Per-role metrics (PRs, lines, cost) available in dashboard |
| Quarterly OKRs defined | Linear goals/OKRs exist for current quarter |

---

## Milestone Dependencies

```
M1 (Foundation) ──┬── M2 (Tools & MCP)
                  ├── M3 (Plugins)
                  │    └── M4 (Agents)
                  └── M7 (Data)
                       └── M8 (Frontend)

M2 ──┬── M5 (Integrations)
     └── M6 (CI/CD)

M5 + M8 ── M9 (Mobile)
M7 + M5 ── M10 (Roadmap)
```

All 10 pass = production-ready jadecli knowledge worker ecosystem.
