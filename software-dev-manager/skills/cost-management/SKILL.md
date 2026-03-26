---
name: cost-management
description: Claude Code budget controls, spend limits, model selection optimization, thinking effort tuning, and cost reduction strategies for orgs.
argument-hint: "<budget|optimize|limits|forecast>"
---

# Cost Management — Budget Controls & Optimization

## Cost Benchmarks

| Metric | Value |
|--------|-------|
| Average cost per developer per day | ~$6 |
| 90th percentile daily cost | <$12 |
| Average monthly cost (Sonnet 4.6) | $100-200/developer |
| Agent teams multiplier | ~7x standard sessions |
| Background token usage | ~$0.04/session |

## Check Session Cost

```
/cost
```

Output: total cost, API duration, wall duration, lines changed.

## Workspace Spend Limits

Set at https://platform.claude.com — workspace-level caps on total Claude Code spend.

## Cost by Model (Supabase query)

```sql
SELECT
  attributes->>'model' AS model,
  COUNT(*)::int AS api_calls,
  ROUND(SUM((attributes->>'cost_usd')::numeric), 2) AS total_cost,
  ROUND(AVG((attributes->>'cost_usd')::numeric), 4) AS avg_cost_per_call
FROM otel_events
WHERE event_name = 'claude_code.api_request'
GROUP BY attributes->>'model'
ORDER BY total_cost DESC;
```

## Cost by User (daily)

```sql
SELECT
  DATE(recorded_at) AS day,
  labels->>'user.email' AS email,
  ROUND(SUM(value)::numeric, 2) AS cost_usd
FROM otel_metrics
WHERE metric_name = 'claude_code.cost.usage'
GROUP BY DATE(recorded_at), labels->>'user.email'
ORDER BY day DESC, cost_usd DESC;
```

## Cost by Team

```sql
SELECT
  labels->>'team.id' AS team,
  labels->>'cost_center' AS cost_center,
  ROUND(SUM(value)::numeric, 2) AS cost_usd
FROM otel_metrics
WHERE metric_name = 'claude_code.cost.usage'
GROUP BY labels->>'team.id', labels->>'cost_center'
ORDER BY cost_usd DESC;
```

## Top Spending Sessions

```sql
SELECT
  attributes->>'session.id' AS session,
  attributes->>'model' AS model,
  ROUND(SUM((attributes->>'cost_usd')::numeric), 2) AS cost,
  SUM((attributes->>'input_tokens')::bigint + (attributes->>'output_tokens')::bigint) AS tokens
FROM otel_events
WHERE event_name = 'claude_code.api_request'
GROUP BY attributes->>'session.id', attributes->>'model'
ORDER BY cost DESC
LIMIT 20;
```

## Optimization Strategies

### Model Selection

| Task | Recommended Model | Why |
|------|------------------|-----|
| Most coding tasks | Sonnet 4.6 | Best cost/capability ratio |
| Complex architecture | Opus 4.6 | Deep reasoning |
| Simple subagent tasks | Haiku 4.5 | Cheapest |
| Switch mid-session | `/model` | No restart needed |

### Thinking Effort

| Setting | When | Command |
|---------|------|---------|
| High | Complex planning/reasoning | `/effort high` |
| Medium (default) | General coding | default |
| Low | Simple tasks | `/effort low` |
| Disabled | Trivial lookups | `/config` → disable thinking |
| Custom budget | Fine control | `MAX_THINKING_TOKENS=8000` |

### Context Reduction

- `/clear` between unrelated tasks
- `/compact` with focus instructions
- Keep CLAUDE.md under 500 lines
- Move specialized instructions to skills (load on-demand)
- Disable unused MCP servers (`/mcp`)
- Use CLI tools (`gh`, `aws`) over MCP servers (less context overhead)
- Delegate verbose operations to subagents

### Agent Teams

- Use Sonnet for teammates (not Opus)
- Keep teams small (each teammate = own context window)
- Keep spawn prompts focused
- Clean up when done — idle teammates still consume tokens

## Budget Alerts (Supabase)

```sql
-- Users over $20/day
SELECT
  labels->>'user.email' AS email,
  DATE(recorded_at) AS day,
  ROUND(SUM(value)::numeric, 2) AS cost_usd
FROM otel_metrics
WHERE metric_name = 'claude_code.cost.usage'
GROUP BY labels->>'user.email', DATE(recorded_at)
HAVING SUM(value) > 20
ORDER BY cost_usd DESC;
```
