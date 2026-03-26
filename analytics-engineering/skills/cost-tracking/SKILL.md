---
name: cost-tracking
description: Track Claude Code token spend and USD cost by model, session, and time range. Budget monitoring and cost-per-agent analysis from Supabase otel_events.
argument-hint: "<summary|by-model|by-session|daily>"
---

# Cost Tracking — Token Spend & USD Analysis

Track costs from `claude_code.token_usage` events in Supabase.

## Queries

### Total Cost Summary

```sql
SELECT
  COUNT(DISTINCT session_id)::int AS session_count,
  SUM((attributes->>'input_tokens')::bigint) AS total_input_tokens,
  SUM((attributes->>'output_tokens')::bigint) AS total_output_tokens,
  SUM((attributes->>'input_tokens')::bigint + (attributes->>'output_tokens')::bigint) AS total_tokens,
  ROUND(SUM((attributes->>'cost_usd')::numeric), 4) AS total_cost_usd,
  ROUND(
    SUM((attributes->>'cost_usd')::numeric) /
    NULLIF(COUNT(DISTINCT session_id), 0), 4
  ) AS avg_cost_per_session
FROM otel_events
WHERE event_name = 'claude_code.token_usage';
```

### Cost by Model

```sql
SELECT
  attributes->>'model' AS model,
  COUNT(DISTINCT session_id)::int AS sessions,
  SUM((attributes->>'input_tokens')::bigint + (attributes->>'output_tokens')::bigint) AS tokens,
  ROUND(SUM((attributes->>'cost_usd')::numeric), 4) AS cost_usd
FROM otel_events
WHERE event_name = 'claude_code.token_usage'
GROUP BY attributes->>'model'
ORDER BY cost_usd DESC;
```

### Daily Cost Trend

```sql
SELECT
  DATE(recorded_at) AS day,
  COUNT(DISTINCT session_id)::int AS sessions,
  SUM((attributes->>'input_tokens')::bigint + (attributes->>'output_tokens')::bigint) AS tokens,
  ROUND(SUM((attributes->>'cost_usd')::numeric), 4) AS cost_usd
FROM otel_events
WHERE event_name = 'claude_code.token_usage'
GROUP BY DATE(recorded_at)
ORDER BY day DESC
LIMIT 30;
```

### Top Spending Sessions

```sql
SELECT
  session_id,
  attributes->>'model' AS model,
  SUM((attributes->>'cost_usd')::numeric) AS total_cost,
  SUM((attributes->>'input_tokens')::bigint + (attributes->>'output_tokens')::bigint) AS total_tokens
FROM otel_events
WHERE event_name = 'claude_code.token_usage'
GROUP BY session_id, attributes->>'model'
ORDER BY total_cost DESC
LIMIT 20;
```

## TypeScript Interface

```typescript
interface CostSummary {
  totalCostUsd: number;
  totalTokens: number;
  totalInputTokens: number;
  totalOutputTokens: number;
  sessionCount: number;
  avgCostPerSession: number;
  costByModel: {
    model: string;
    cost: number;
    tokens: number;
    sessions: number;
  }[];
}
```

## Budget Alerts

Set thresholds and query for sessions exceeding them:

```sql
-- Sessions over $5
SELECT session_id, SUM((attributes->>'cost_usd')::numeric) AS cost
FROM otel_events
WHERE event_name = 'claude_code.token_usage'
GROUP BY session_id
HAVING SUM((attributes->>'cost_usd')::numeric) > 5
ORDER BY cost DESC;
```
