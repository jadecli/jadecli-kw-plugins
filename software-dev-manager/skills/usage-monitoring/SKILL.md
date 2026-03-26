---
name: usage-monitoring
description: Track Claude Code token usage, sessions, tool calls, lines of code, commits, and PRs per user and team. Supabase OTel queries for usage dashboards.
argument-hint: "<tokens|sessions|tools|productivity|by-user|by-team>"
---

# Usage Monitoring — Token, Session & Productivity Tracking

## Token Usage by Model

```sql
SELECT
  labels->>'model' AS model,
  labels->>'type' AS token_type,
  SUM(value)::bigint AS tokens
FROM otel_metrics
WHERE metric_name = 'claude_code.token.usage'
GROUP BY labels->>'model', labels->>'type'
ORDER BY tokens DESC;
```

## Token Usage by User

```sql
SELECT
  labels->>'user.email' AS email,
  labels->>'model' AS model,
  SUM(CASE WHEN labels->>'type' = 'input' THEN value ELSE 0 END)::bigint AS input_tokens,
  SUM(CASE WHEN labels->>'type' = 'output' THEN value ELSE 0 END)::bigint AS output_tokens,
  SUM(CASE WHEN labels->>'type' = 'cacheRead' THEN value ELSE 0 END)::bigint AS cache_read
FROM otel_metrics
WHERE metric_name = 'claude_code.token.usage'
GROUP BY labels->>'user.email', labels->>'model'
ORDER BY input_tokens + output_tokens DESC;
```

## Daily Active Users & Sessions

```sql
SELECT
  DATE(recorded_at) AS day,
  COUNT(DISTINCT labels->>'user.account_uuid') AS dau,
  SUM(value)::int AS sessions
FROM otel_metrics
WHERE metric_name = 'claude_code.session.count'
GROUP BY DATE(recorded_at)
ORDER BY day DESC
LIMIT 30;
```

## Lines of Code by User

```sql
SELECT
  labels->>'user.email' AS email,
  SUM(CASE WHEN labels->>'type' = 'added' THEN value ELSE 0 END)::int AS lines_added,
  SUM(CASE WHEN labels->>'type' = 'removed' THEN value ELSE 0 END)::int AS lines_removed
FROM otel_metrics
WHERE metric_name = 'claude_code.lines_of_code.count'
GROUP BY labels->>'user.email'
ORDER BY lines_added DESC;
```

## Commits & PRs by User

```sql
SELECT
  labels->>'user.email' AS email,
  SUM(CASE WHEN metric_name = 'claude_code.commit.count' THEN value ELSE 0 END)::int AS commits,
  SUM(CASE WHEN metric_name = 'claude_code.pull_request.count' THEN value ELSE 0 END)::int AS prs
FROM otel_metrics
WHERE metric_name IN ('claude_code.commit.count', 'claude_code.pull_request.count')
GROUP BY labels->>'user.email'
ORDER BY commits DESC;
```

## Tool Usage Breakdown

```sql
SELECT
  attributes->>'tool_name' AS tool,
  COUNT(*)::int AS calls,
  COUNT(*) FILTER (WHERE (attributes->>'success')::boolean) AS successes,
  ROUND(AVG((attributes->>'duration_ms')::float)::numeric, 1) AS avg_ms
FROM otel_events
WHERE event_name = 'claude_code.tool_result'
GROUP BY attributes->>'tool_name'
ORDER BY calls DESC;
```

## Active Time by User

```sql
SELECT
  labels->>'user.email' AS email,
  SUM(CASE WHEN labels->>'type' = 'user' THEN value ELSE 0 END)::int AS user_time_sec,
  SUM(CASE WHEN labels->>'type' = 'cli' THEN value ELSE 0 END)::int AS cli_time_sec,
  SUM(value)::int AS total_active_sec
FROM otel_metrics
WHERE metric_name = 'claude_code.active_time.total'
GROUP BY labels->>'user.email'
ORDER BY total_active_sec DESC;
```

## By Team (via OTEL_RESOURCE_ATTRIBUTES)

```sql
SELECT
  labels->>'team.id' AS team,
  SUM(value)::bigint AS tokens
FROM otel_metrics
WHERE metric_name = 'claude_code.token.usage'
GROUP BY labels->>'team.id'
ORDER BY tokens DESC;
```

## Key Metrics Summary

| Metric | What It Tells You |
|--------|-------------------|
| Token usage by model | Which models consume most budget |
| DAU/sessions | Adoption and engagement trends |
| Lines of code | Productivity per user |
| Commits & PRs | Development velocity |
| Tool success rates | Tool reliability and error patterns |
| Active time | Actual working time vs idle |
