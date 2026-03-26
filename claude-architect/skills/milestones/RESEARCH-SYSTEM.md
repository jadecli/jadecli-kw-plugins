# jadecli Multi-Agent Research System

## Design: Every Department Runs Its Own Research System

Each knowledge worker plugin operates as an **independent multi-agent research system** (per Anthropic's architecture). Departments produce structured outputs that feed the architect as inputs. The architect uses these to parallelize work across milestones.

```
┌─────────────────────────────────────────────────────────┐
│                   ARCHITECT (Lead Agent)                  │
│  Receives structured research from all departments       │
│  Updates milestone plan, resolves cross-dept blockers    │
│  Parallelizes work via fork_session + Agent spawning     │
└────────┬──────┬──────┬──────┬──────┬──────┬─────────────┘
         │      │      │      │      │      │
    ┌────▼─┐ ┌─▼───┐ ┌▼────┐ ┌▼────┐ ┌▼───┐ ┌▼────┐
    │Data  │ │Front│ │Integ│ │Analy│ │SDM │ │QAE  │
    │Eng   │ │End  │ │rati │ │tics │ │    │ │     │
    │      │ │Eng  │ │on   │ │Eng  │ │    │ │     │
    └──┬───┘ └──┬──┘ └──┬──┘ └──┬──┘ └─┬──┘ └──┬──┘
       │        │       │       │      │       │
    3-5 sub  3-5 sub 3-5 sub 3-5 sub 3-5 sub 3-5 sub
    agents   agents  agents  agents  agents  agents
```

## Department Research System Template

Each department follows the same orchestrator-worker pattern from Anthropic's multi-agent research system. The lead agent is the plugin's primary skill. Subagents are spawned per research question.

### Structured Research Input (from Architect → Department)

```json
{
  "department": "data-engineering",
  "research_question": "What blocking dependencies exist between M7 (Data) conditions?",
  "context": {
    "milestone": "M7",
    "blocking_conditions": ["otel_events has real sessions > 10", "Cube.js models queryable"],
    "current_state": "OTel pipeline connected (M1 passed), but no real session data yet"
  },
  "effort_level": "medium",
  "expected_output_schema": {
    "type": "object",
    "properties": {
      "findings": { "type": "array", "items": { "$ref": "#/definitions/finding" } },
      "blockers": { "type": "array", "items": { "type": "string" } },
      "recommendations": { "type": "array", "items": { "type": "string" } },
      "estimated_effort": { "type": "string", "enum": ["hours", "days", "sprint"] },
      "dependencies_on_other_departments": { "type": "array", "items": { "$ref": "#/definitions/dependency" } }
    },
    "required": ["findings", "blockers", "recommendations"]
  }
}
```

### Structured Research Output (Department → Architect)

```json
{
  "department": "data-engineering",
  "milestone": "M7",
  "timestamp": "2026-03-26T12:00:00Z",
  "findings": [
    {
      "id": "DE-F001",
      "title": "OTel Collector not forwarding to Supabase",
      "severity": "P1",
      "evidence": "SELECT COUNT(*) FROM otel_events returns 0",
      "tools_used": ["Bash(psql)", "Read(otel-collector-config.yaml)"]
    }
  ],
  "blockers": [
    "OTEL_EXPORTER_OTLP_ENDPOINT not configured in managed settings",
    "Supabase otel_events table missing GIN index on attributes"
  ],
  "recommendations": [
    "Deploy managed settings with OTEL_EXPORTER_OTLP_ENDPOINT pointing to Supabase",
    "Run schema/001_otel_events.sql migration"
  ],
  "estimated_effort": "hours",
  "dependencies_on_other_departments": [
    {
      "department": "integration-engineering",
      "what": "OTel collector deployment config",
      "blocks": "M7: otel_events populated"
    },
    {
      "department": "frontend-engineering",
      "what": "Netlify env vars for SUPABASE_URL",
      "blocks": "M8: dashboard renders real data"
    }
  ]
}
```

## Department-Specific Research Configurations

### data-engineering (Lead: Opus, Subagents: Sonnet)

| Research Area | Subagent Count | Tools | Canonical Tools Used |
|---------------|---------------|-------|---------------------|
| Package drift audit | 1 | `Bash(npm audit)`, `Read(versions.json)`, `Glob(**/package.json)` | Glob, Read, Bash |
| AST index freshness | 1 | `Bash(python scripts/build-ast-index.py)`, `Read(ast-index/*.json)` | Bash, Read |
| Supabase schema validation | 2 | `Bash(psql)`, `Read(schema/*.sql)`, `Grep(GRAIN)` | Bash, Read, Grep |
| Cube.js model ↔ schema sync | 1 | `Read(cube/model/*.yml)`, `Read(schema/**/*.sql)` | Read, Glob |

**Effort scaling**: Simple fact-finding (1 agent, 3-10 tool calls) → Schema validation (2 agents, 10-15 calls)

### frontend-engineering (Lead: Sonnet, Subagents: Haiku)

| Research Area | Subagent Count | Tools | Canonical Tools Used |
|---------------|---------------|-------|---------------------|
| Netlify deploy status | 1 | `Bash(netlify status)`, `WebFetch(jadecli.com)` | Bash, WebFetch |
| Framework config audit | 2 | `Read(next.config.ts)`, `Read(vercel.json)`, `Glob(**/*.config.*)` | Read, Glob |
| Edge function validation | 1 | `Bash(netlify functions:list)` | Bash |

### integration-engineering (Lead: Opus, Subagents: Sonnet)

| Research Area | Subagent Count | Tools | Canonical Tools Used |
|---------------|---------------|-------|---------------------|
| Channel connectivity | 3 | `Bash(claude --channels --help)`, MCP tools | Bash |
| GitHub Actions status | 2 | `Bash(gh run list)`, `Bash(gh api)` | Bash |
| Slack integration | 1 | `WebFetch`, `Bash(gh api)` | WebFetch, Bash |
| Remote Control test | 1 | `Bash(claude remote-control --help)` | Bash |

### analytics-engineering (Lead: Sonnet, Subagents: Haiku)

| Research Area | Subagent Count | Tools | Canonical Tools Used |
|---------------|---------------|-------|---------------------|
| OTel event volume | 1 | `Bash(psql)` against Supabase | Bash |
| Cost summary | 1 | `Bash(psql)` aggregate queries | Bash |
| Tool success rates | 1 | `Bash(psql)` tool_result analysis | Bash |
| Dashboard data freshness | 1 | `WebFetch(jadecli.com/telemetry)` | WebFetch |

### software-dev-manager (Lead: Opus, Subagents: Sonnet)

| Research Area | Subagent Count | Tools | Canonical Tools Used |
|---------------|---------------|-------|---------------------|
| Org adoption metrics | 2 | `Bash(psql)` DAU/WAU/MAU, `Bash(gh api)` PR attribution | Bash |
| Cost trends | 1 | `Bash(psql)` daily cost queries | Bash |
| Rate limit headroom | 1 | `Bash(psql)` peak TPM analysis | Bash |
| KW ROI per role | 2 | `Bash(psql)` cross-role comparison | Bash |

### claude-qae (Lead: Opus, Subagents: Sonnet)

| Research Area | Subagent Count | Tools | Canonical Tools Used |
|---------------|---------------|-------|---------------------|
| Test coverage gaps | 2 | `Bash(npx vitest --coverage)`, `Glob(**/*.test.*)` | Bash, Glob |
| CI pipeline health | 1 | `Bash(gh run list)` | Bash |
| Anti-pattern scan | 3 | `Grep(ANTHROPIC_API_KEY)`, `Grep(isError)`, `Grep(any)` | Grep |
| Milestone condition check | 5+ | Full milestone test suite | All canonical tools |

## Architect Parallelization Strategy

The architect receives all department research outputs and uses them to parallelize milestone work:

### Phase 1: Foundation (M1-M2) — Sequential, blocking

```
M1 conditions must pass before anything else.
Architect assigns:
  - data-engineering: Supabase schema + OTel pipeline
  - integration-engineering: OTel collector config
  - QAE: foundation tests

Canonical tools: Agent (spawn dept subagents), TaskCreate (track), Read (results)
```

### Phase 2: Plugins + Data (M3, M4, M7) — Parallel

```
Once M1 passes, three milestones can run in parallel:
  fork_session → M3 branch (all plugins install)
  fork_session → M4 branch (agent teams pass tests)
  fork_session → M7 branch (real data in Supabase)

Architect spawns 3 Agent subagents simultaneously.
Each gets scoped tools: only what their milestone needs.
```

### Phase 3: Integrations + CI (M5, M6) — Parallel, depends on M2

```
Once M2 (MCP) passes:
  fork_session → M5 branch (Slack, Channels, RC, Dispatch)
  fork_session → M6 branch (GitHub Actions, code review)

Architect spawns 2 Agent subagents.
integration-engineering leads M5; QAE validates M6.
```

### Phase 4: Frontend + Mobile (M8, M9) — Parallel, depends on M5+M7

```
Once M5 and M7 both pass:
  fork_session → M8 branch (Netlify + dashboard with real data)
  fork_session → M9 branch (iOS mobile E2E)

frontend-engineering leads M8; integration-engineering leads M9.
```

### Phase 5: Roadmap (M10) — Final, depends on M7+M5

```
Once M7 and M5 pass:
  M10: Linear integration + WBR generation
  software-dev-manager leads.
```

### Dependency Graph with Parallelization

```
         M1 ──────────────────────────────┐
         │                                │
    ┌────▼────┐                      ┌────▼────┐
    │   M2    │                      │   M7    │
    │ MCP     │                      │ Data    │
    └────┬────┘                      └────┬────┘
         │                                │
    ┌────▼────┬────────┐            ┌────▼────┐
    │   M5    │   M6   │            │   M3    │
    │ Integr  │ CI/CD  │            │ Plugins │
    └────┬────┘────────┘            └────┬────┘
         │                               │
    ┌────▼────┐                     ┌────▼────┐
    │   M9    │                     │   M4    │
    │ Mobile  │                     │ Agents  │
    └─────────┘                     └─────────┘
         │
    ┌────▼────┐
    │   M8    │ ← also needs M7
    │Frontend │
    └────┬────┘
         │
    ┌────▼────┐
    │  M10    │ ← also needs M5
    │Roadmap  │
    └─────────┘

Max parallel width: 3 (Phase 2: M3 + M4 + M7)
Critical path: M1 → M2 → M5 → M9 → M8 → M10
```

## Canonical Claude Code Tools Used

Per Anthropic's tools reference, these are the exact tool names used in department research:

| Tool | Permission | Department Usage |
|------|-----------|-----------------|
| `Agent` | No | Architect spawns department leads; departments spawn subagents |
| `Read` | No | All departments read configs, schemas, test results |
| `Grep` | No | QAE scans for anti-patterns; data-eng searches schemas |
| `Glob` | No | All departments find files by pattern |
| `Bash` | Yes | All departments execute commands (psql, gh, npm, vitest) |
| `Edit` | Yes | Departments fix issues found by research |
| `Write` | Yes | Create new configs, schemas, tests |
| `WebFetch` | Yes | Frontend checks jadecli.com; integration checks Slack |
| `WebSearch` | Yes | Research external docs when needed |
| `TaskCreate` | No | Architect tracks milestone progress |
| `TaskUpdate` | No | Departments report completion |
| `CronCreate` | No | Schedule recurring research (nightly AST rebuild, weekly audit) |
| `EnterPlanMode` | No | Architect plans before parallelizing |
| `ExitPlanMode` | Yes | Architect presents plan for approval |
| `ToolSearch` | No | Find deferred MCP tools on-demand |
| `Skill` | Yes | Invoke department-specific skills |

## Advanced Tool Use Patterns Applied

### Tool Search (85% token reduction)

Keep 3-5 most-used tools always loaded per department. Defer the rest:
- data-engineering always: `Bash`, `Read`, `Glob`, `Grep`
- data-engineering deferred: `WebFetch`, `WebSearch`, `NotebookEdit`

### Programmatic Tool Calling (37% token reduction)

Department subagents use code execution for multi-step data processing:
```python
# analytics-engineering subagent: aggregate OTel data
sessions = await bash("psql $SUPABASE_DB_URL -t -c 'SELECT ...'")
tool_calls = await bash("psql $SUPABASE_DB_URL -t -c 'SELECT ...'")
cost = await bash("psql $SUPABASE_DB_URL -t -c 'SELECT ...'")
# Only final summary returns to coordinator
return {"sessions": int(sessions), "tool_calls": int(tool_calls), "cost": float(cost)}
```

### Tool Use Examples (72% → 90% accuracy)

Each MCP tool includes input_examples showing correct invocation:
```json
{
  "name": "supabase:query",
  "input_examples": [
    {
      "sql": "SELECT COUNT(*) FROM otel_events WHERE event_name = 'claude_code.session.stop' AND recorded_at > now() - INTERVAL '24 hours'",
      "description": "Count sessions in last 24 hours"
    }
  ]
}
```

## Output: Architect's Parallelized Plan

After all departments report, the architect produces:

```json
{
  "plan_version": "2026-03-26-v1",
  "milestones_status": {
    "M1": "passed",
    "M2": "in_progress",
    "M3": "blocked_on_M1",
    "M4": "blocked_on_M3",
    "M5": "blocked_on_M2",
    "M6": "blocked_on_M2",
    "M7": "in_progress",
    "M8": "blocked_on_M5_M7",
    "M9": "blocked_on_M5_M8",
    "M10": "blocked_on_M5_M7"
  },
  "parallel_work_items": [
    {"milestone": "M2", "department": "integration-engineering", "task": "Configure MCP servers"},
    {"milestone": "M7", "department": "data-engineering", "task": "Deploy OTel schema to Supabase"}
  ],
  "cross_department_blockers": [
    {
      "blocker": "SUPABASE_URL not in Netlify env",
      "blocking_milestone": "M8",
      "owning_department": "frontend-engineering",
      "depends_on": "data-engineering (Supabase project created)"
    }
  ],
  "next_actions": [
    "data-engineering: run schema migration",
    "integration-engineering: deploy OTel collector config",
    "QAE: run M1 milestone tests to validate"
  ]
}
```

## References

- [Anthropic Multi-Agent Research System](https://www.anthropic.com/engineering/multi-agent-research-system)
- [Advanced Tool Use](https://www.anthropic.com/engineering/advanced-tool-use)
- [Tools Reference](https://code.claude.com/docs/en/tools-reference)
- [Claude Certified Architect Foundations](https://anthropic.skilljar.com/claude-certified-architect-foundations-access-request)
- architecture-playbook.pdf — Enterprise LLM Architecture Playbook
- architecture-cert.pdf — Exam guide with 5 domains, 6 scenarios
