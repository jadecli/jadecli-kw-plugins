---
name: knowledge-worker-metrics
description: Per-role analytics for jadecli knowledge worker plugins — measure ROI across data-engineering, frontend-engineering, integration-engineering, and analytics-engineering roles.
argument-hint: "<by-role|by-plugin|roi|compare>"
---

# Knowledge Worker Metrics — Per-Role Analytics

Measure how each knowledge worker plugin drives productivity across the org.

## Knowledge Worker Roles

| Plugin | Role | Key Outputs |
|--------|------|-------------|
| `data-engineering` | Data engineers | Package audits, AST indexes, Cube.js models, scheduled tasks |
| `frontend-engineering` | Frontend engineers | Netlify deploys, edge functions, framework configs |
| `integration-engineering` | Integration engineers | Channels, Slack sessions, GitHub Actions, remote control |
| `analytics-engineering` | Analytics engineers | OTel queries, dashboards, cost reports, Kimball models |
| `software-dev-manager` | Dev managers | Org analytics, budget controls, capacity planning |

## Track Plugin Usage (via tool events)

When a skill is invoked, it appears in `tool_result` events with `skill_name` attribute (when `OTEL_LOG_TOOL_DETAILS=1`):

```sql
-- Skill invocations by plugin
SELECT
  SPLIT_PART(attributes->>'skill_name', ':', 1) AS plugin,
  attributes->>'skill_name' AS skill,
  COUNT(*)::int AS invocations,
  COUNT(DISTINCT session_id) AS unique_sessions
FROM otel_events
WHERE event_name = 'claude_code.tool_result'
  AND attributes->>'skill_name' IS NOT NULL
GROUP BY plugin, skill
ORDER BY invocations DESC;
```

## Productivity by Role

Map users to roles via `OTEL_RESOURCE_ATTRIBUTES`:

```bash
# Set per developer
export OTEL_RESOURCE_ATTRIBUTES="team.id=platform,role=data-engineer,cost_center=eng-123"
```

Then query:

```sql
SELECT
  labels->>'role' AS role,
  COUNT(DISTINCT labels->>'user.account_uuid') AS users,
  SUM(CASE WHEN metric_name = 'claude_code.pull_request.count' THEN value ELSE 0 END)::int AS prs,
  SUM(CASE WHEN metric_name = 'claude_code.lines_of_code.count' THEN value ELSE 0 END)::int AS lines,
  ROUND(SUM(CASE WHEN metric_name = 'claude_code.cost.usage' THEN value ELSE 0 END)::numeric, 2) AS cost_usd,
  ROUND(
    SUM(CASE WHEN metric_name = 'claude_code.pull_request.count' THEN value ELSE 0 END) /
    NULLIF(COUNT(DISTINCT labels->>'user.account_uuid'), 0)::numeric, 1
  ) AS prs_per_user
FROM otel_metrics
WHERE metric_name IN ('claude_code.pull_request.count', 'claude_code.lines_of_code.count', 'claude_code.cost.usage')
GROUP BY labels->>'role'
ORDER BY prs DESC;
```

## ROI per Role

| Role | Cost Signal | Value Signal | ROI Formula |
|------|------------|--------------|-------------|
| Data engineer | Tokens + model cost | AST indexes built, packages audited, pipelines scheduled | (hours_saved × hourly_rate) / token_cost |
| Frontend engineer | Tokens + model cost | Deploys, edge functions, framework configs | deploy_frequency_increase / token_cost |
| Integration engineer | Tokens + model cost | Channels set up, Actions configured, remote sessions | automation_hours_saved / token_cost |
| Analytics engineer | Tokens + model cost | Dashboards built, queries written, models defined | reporting_time_saved / token_cost |

## Cross-Role Comparison

```sql
SELECT
  labels->>'role' AS role,
  ROUND(SUM(CASE WHEN metric_name = 'claude_code.cost.usage' THEN value ELSE 0 END)::numeric, 2) AS total_cost,
  SUM(CASE WHEN metric_name = 'claude_code.lines_of_code.count' THEN value ELSE 0 END)::int AS total_lines,
  ROUND(
    SUM(CASE WHEN metric_name = 'claude_code.cost.usage' THEN value ELSE 0 END) /
    NULLIF(SUM(CASE WHEN metric_name = 'claude_code.lines_of_code.count' THEN value ELSE 0 END), 0)::numeric, 4
  ) AS cost_per_line
FROM otel_metrics
WHERE metric_name IN ('claude_code.cost.usage', 'claude_code.lines_of_code.count')
GROUP BY labels->>'role'
ORDER BY cost_per_line ASC;
```

## Adoption Funnel by Role

| Stage | Metric | Target |
|-------|--------|--------|
| Awareness | Plugin installed | 100% of role |
| Activation | First skill invocation | 80% within 7 days |
| Engagement | Daily active sessions | 60% DAU |
| Retention | Weekly active users | 70% WAU |
| Advocacy | Suggestion accept rate >80% | 50% of role |

## Weekly Business Review (WBR) Template

```
# Knowledge Worker WBR — Week of YYYY-MM-DD

## Headlines
- DAU: X (+Y% WoW)
- Total cost: $X (-Y% WoW)
- PRs with Claude Code: X% of all merged PRs

## By Role
| Role | Users | PRs | Lines | Cost | Cost/PR |
|------|-------|-----|-------|------|---------|

## Highlights
- [role] shipped [X] using [plugin:skill]

## Actions
- [ ] Action item from metrics
```
