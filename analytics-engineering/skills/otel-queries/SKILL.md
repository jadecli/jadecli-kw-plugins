---
name: otel-queries
description: Raw SQL queries against Supabase otel_events and otel_metrics tables. Custom telemetry analysis for Claude Code sessions.
argument-hint: "<query-description>"
---

# OTel Queries — Custom Telemetry Analysis

Run SQL against Supabase `otel_events` and `otel_metrics` for ad-hoc analysis.

## Schema

### otel_events

```sql
CREATE TABLE otel_events (
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

### otel_metrics

```sql
CREATE TABLE otel_metrics (
  id BIGSERIAL PRIMARY KEY,
  metric_name TEXT NOT NULL,
  value DOUBLE PRECISION NOT NULL,
  labels JSONB DEFAULT '{}',
  recorded_at TIMESTAMPTZ DEFAULT now()
);
```

## Event Types

| event_name | attributes keys |
|------------|----------------|
| `claude_code.session.stop` | agent_type |
| `claude_code.tool_result` | tool_name, success, duration_ms |
| `claude_code.token_usage` | model, input_tokens, output_tokens, cost_usd |

## Common Queries

### Sessions in the last 24 hours

```sql
SELECT session_id, attributes->>'agent_type' AS agent_type, recorded_at
FROM otel_events
WHERE event_name = 'claude_code.session.stop'
  AND recorded_at > now() - INTERVAL '24 hours'
ORDER BY recorded_at DESC;
```

### Tool calls per session

```sql
SELECT session_id, COUNT(*) AS tool_calls
FROM otel_events
WHERE event_name = 'claude_code.tool_result'
GROUP BY session_id
ORDER BY tool_calls DESC
LIMIT 20;
```

### Failing tools

```sql
SELECT
  attributes->>'tool_name' AS tool,
  COUNT(*) FILTER (WHERE NOT (attributes->>'success')::boolean) AS failures,
  COUNT(*) AS total,
  ROUND(
    AVG(CASE WHEN (attributes->>'success')::boolean THEN 1 ELSE 0 END)::numeric, 3
  ) AS success_rate
FROM otel_events
WHERE event_name = 'claude_code.tool_result'
GROUP BY attributes->>'tool_name'
HAVING COUNT(*) FILTER (WHERE NOT (attributes->>'success')::boolean) > 0
ORDER BY failures DESC;
```

### Daily session counts

```sql
SELECT
  DATE(recorded_at) AS day,
  COUNT(*) AS sessions
FROM otel_events
WHERE event_name = 'claude_code.session.stop'
GROUP BY DATE(recorded_at)
ORDER BY day DESC
LIMIT 30;
```

### Slowest tool calls

```sql
SELECT
  attributes->>'tool_name' AS tool,
  (attributes->>'duration_ms')::float AS duration_ms,
  session_id,
  recorded_at
FROM otel_events
WHERE event_name = 'claude_code.tool_result'
  AND attributes->>'duration_ms' IS NOT NULL
ORDER BY (attributes->>'duration_ms')::float DESC
LIMIT 20;
```

## Via Supabase Client

```typescript
const { data, error } = await supabase
  .from("otel_events")
  .select("session_id, attributes, recorded_at")
  .eq("event_name", "claude_code.session.stop")
  .order("recorded_at", { ascending: false })
  .limit(50);
```
