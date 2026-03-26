# Analytics Engineering Plugin

Telemetry analytics for Claude Code agent sessions. Built on the OTel event pipeline from jadecli/jadebot PR #222, adapted for Supabase PostgreSQL.

Queries `otel_events` and `otel_metrics` tables to surface session activity, tool performance, cost tracking, and error rates.

## Skills

| Skill | Description |
|-------|-------------|
| `telemetry-dashboard` | Dashboard with session stats, tool breakdown, and cost cards |
| `otel-queries` | SQL queries against Supabase otel_events for custom analysis |
| `cost-tracking` | Token spend and USD cost by model, session, and time range |
| `tool-analytics` | Tool call success rates, duration, and error patterns |
| `session-analytics` | Agent session history, duration, tool counts, and agent types |
| `kimball-modeling` | Kimball dimensional modeling — semantic layer, 1-table-per-file DDL, load types, pipeline metadata, version-controlled business logic |

## Data Model

Events stored in Supabase PostgreSQL via OpenTelemetry:

| Table | Key Columns |
|-------|-------------|
| `otel_events` | event_name, session_id, attributes (JSONB), recorded_at |
| `otel_metrics` | metric_name, value, labels (JSONB), recorded_at |

### Event Types

| event_name | attributes |
|------------|-----------|
| `claude_code.session.stop` | agent_type |
| `claude_code.tool_result` | tool_name, success, duration_ms |
| `claude_code.token_usage` | model, input_tokens, output_tokens, cost_usd |

## Requires

- `SUPABASE_URL` and `SUPABASE_SERVICE_ROLE_KEY` environment variables
- `@supabase/supabase-js` package
