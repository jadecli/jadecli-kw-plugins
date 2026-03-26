---
name: session-analytics
description: Analyze Claude Code agent sessions — history, duration, tool counts, agent types, and session patterns from Supabase otel_events.
argument-hint: "<recent|by-agent|longest|summary>"
---

# Session Analytics — Agent Session History

Analyze `claude_code.session.stop` events from Supabase.

## Queries

### Recent Sessions

```sql
SELECT
  e.session_id,
  e.attributes->>'agent_type' AS agent_type,
  e.recorded_at,
  COALESCE(t.tool_count, 0)::int AS tool_count
FROM otel_events e
LEFT JOIN (
  SELECT session_id, COUNT(*)::int AS tool_count
  FROM otel_events
  WHERE event_name = 'claude_code.tool_result'
  GROUP BY session_id
) t ON t.session_id = e.session_id
WHERE e.event_name = 'claude_code.session.stop'
ORDER BY e.recorded_at DESC
LIMIT 50;
```

### Summary Stats

```sql
SELECT
  (SELECT COUNT(*)::int FROM otel_events WHERE event_name = 'claude_code.session.stop') AS total_sessions,
  (SELECT COUNT(*)::int FROM otel_events WHERE event_name = 'claude_code.tool_result') AS total_tool_calls,
  (SELECT ROUND(
    AVG(CASE WHEN (attributes->>'success')::boolean THEN 0 ELSE 1 END)::numeric, 4
  )::float FROM otel_events WHERE event_name = 'claude_code.tool_result') AS error_rate;
```

### Sessions by Agent Type

```sql
SELECT
  attributes->>'agent_type' AS agent_type,
  COUNT(*)::int AS sessions,
  MIN(recorded_at) AS first_seen,
  MAX(recorded_at) AS last_seen
FROM otel_events
WHERE event_name = 'claude_code.session.stop'
GROUP BY attributes->>'agent_type'
ORDER BY sessions DESC;
```

### Sessions with Most Tool Calls

```sql
SELECT
  e.session_id,
  e.attributes->>'agent_type' AS agent_type,
  e.recorded_at,
  t.tool_count
FROM otel_events e
JOIN (
  SELECT session_id, COUNT(*)::int AS tool_count
  FROM otel_events
  WHERE event_name = 'claude_code.tool_result'
  GROUP BY session_id
) t ON t.session_id = e.session_id
WHERE e.event_name = 'claude_code.session.stop'
ORDER BY t.tool_count DESC
LIMIT 20;
```

### Daily Session Volume

```sql
SELECT
  DATE(recorded_at) AS day,
  COUNT(*)::int AS sessions,
  COUNT(DISTINCT attributes->>'agent_type') AS unique_agents
FROM otel_events
WHERE event_name = 'claude_code.session.stop'
GROUP BY DATE(recorded_at)
ORDER BY day DESC
LIMIT 30;
```

### Session with Tool + Cost Detail

```sql
SELECT
  s.session_id,
  s.attributes->>'agent_type' AS agent_type,
  s.recorded_at,
  COALESCE(t.tool_count, 0) AS tool_calls,
  COALESCE(t.errors, 0) AS tool_errors,
  COALESCE(c.cost_usd, 0) AS cost_usd,
  COALESCE(c.tokens, 0) AS tokens
FROM otel_events s
LEFT JOIN (
  SELECT session_id,
    COUNT(*)::int AS tool_count,
    COUNT(*) FILTER (WHERE NOT (attributes->>'success')::boolean)::int AS errors
  FROM otel_events WHERE event_name = 'claude_code.tool_result'
  GROUP BY session_id
) t ON t.session_id = s.session_id
LEFT JOIN (
  SELECT session_id,
    ROUND(SUM((attributes->>'cost_usd')::numeric), 4) AS cost_usd,
    SUM((attributes->>'input_tokens')::bigint + (attributes->>'output_tokens')::bigint) AS tokens
  FROM otel_events WHERE event_name = 'claude_code.token_usage'
  GROUP BY session_id
) c ON c.session_id = s.session_id
WHERE s.event_name = 'claude_code.session.stop'
ORDER BY s.recorded_at DESC
LIMIT 30;
```

## Supabase Migration

```sql
-- Run once to create the otel_events table
CREATE TABLE IF NOT EXISTS otel_events (
  id BIGSERIAL PRIMARY KEY,
  event_name TEXT NOT NULL,
  session_id TEXT,
  attributes JSONB DEFAULT '{}',
  recorded_at TIMESTAMPTZ DEFAULT now()
);

CREATE INDEX idx_otel_events_name ON otel_events(event_name);
CREATE INDEX idx_otel_events_session ON otel_events(session_id);
CREATE INDEX idx_otel_events_recorded ON otel_events(recorded_at);
CREATE INDEX idx_otel_events_attrs ON otel_events USING gin(attributes);
ALTER TABLE otel_events ENABLE ROW LEVEL SECURITY;
```
