---
name: otel-setup
description: Configure OpenTelemetry export from Claude Code to Supabase. Managed settings for org-wide rollout, collector config, and event schema.
argument-hint: "<init|managed-settings|collector|schema>"
---

# OTel Setup — Foundational Telemetry Pipeline

## Quick Start (per developer)

```bash
export CLAUDE_CODE_ENABLE_TELEMETRY=1
export OTEL_METRICS_EXPORTER=otlp
export OTEL_LOGS_EXPORTER=otlp
export OTEL_EXPORTER_OTLP_PROTOCOL=grpc
export OTEL_EXPORTER_OTLP_ENDPOINT=http://collector.your-org.com:4317
export OTEL_EXPORTER_OTLP_HEADERS="Authorization=Bearer your-token"
```

## Org-Wide Managed Settings

Distribute via MDM or managed settings file — users cannot override:

```json
{
  "env": {
    "CLAUDE_CODE_ENABLE_TELEMETRY": "1",
    "OTEL_METRICS_EXPORTER": "otlp",
    "OTEL_LOGS_EXPORTER": "otlp",
    "OTEL_EXPORTER_OTLP_PROTOCOL": "grpc",
    "OTEL_EXPORTER_OTLP_ENDPOINT": "http://collector.jadecli.com:4317",
    "OTEL_EXPORTER_OTLP_HEADERS": "Authorization=Bearer org-token"
  }
}
```

## Multi-Team Resource Attributes

```bash
export OTEL_RESOURCE_ATTRIBUTES="department=engineering,team.id=platform,cost_center=eng-123"
```

Filter dashboards by team, department, cost center.

## Metrics Exported

| Metric | Unit | Attributes |
|--------|------|-----------|
| `claude_code.session.count` | count | session.id, user.account_uuid |
| `claude_code.token.usage` | tokens | type (input/output/cacheRead), model |
| `claude_code.cost.usage` | USD | model |
| `claude_code.lines_of_code.count` | count | type (added/removed) |
| `claude_code.commit.count` | count | — |
| `claude_code.pull_request.count` | count | — |
| `claude_code.active_time.total` | seconds | type (user/cli) |
| `claude_code.code_edit_tool.decision` | count | tool_name, decision, language |

## Events Exported

| Event | Key Attributes |
|-------|---------------|
| `claude_code.user_prompt` | prompt_length, prompt.id |
| `claude_code.tool_result` | tool_name, success, duration_ms |
| `claude_code.api_request` | model, cost_usd, input_tokens, output_tokens, cache_read_tokens, speed |
| `claude_code.api_error` | model, error, status_code, attempt |
| `claude_code.tool_decision` | tool_name, decision, source |

## Standard Attributes (all signals)

| Attribute | Description |
|-----------|-------------|
| `session.id` | Unique session ID |
| `organization.id` | Org UUID |
| `user.account_uuid` | User UUID |
| `user.account_id` | User ID (tagged format) |
| `user.email` | User email (OAuth) |
| `terminal.type` | iTerm, vscode, cursor, tmux |

## Supabase Schema

```sql
CREATE TABLE otel_events (
  id BIGSERIAL PRIMARY KEY,
  event_name TEXT NOT NULL,
  session_id TEXT,
  attributes JSONB DEFAULT '{}',
  recorded_at TIMESTAMPTZ DEFAULT now()
);

CREATE TABLE otel_metrics (
  id BIGSERIAL PRIMARY KEY,
  metric_name TEXT NOT NULL,
  value DOUBLE PRECISION NOT NULL,
  labels JSONB DEFAULT '{}',
  recorded_at TIMESTAMPTZ DEFAULT now()
);

CREATE INDEX idx_otel_events_name ON otel_events(event_name);
CREATE INDEX idx_otel_events_session ON otel_events(session_id);
CREATE INDEX idx_otel_events_recorded ON otel_events(recorded_at);
CREATE INDEX idx_otel_events_attrs ON otel_events USING gin(attributes);
ALTER TABLE otel_events ENABLE ROW LEVEL SECURITY;
```

## Cardinality Control

| Variable | Default | Effect |
|----------|---------|--------|
| `OTEL_METRICS_INCLUDE_SESSION_ID` | true | Disable for lower cardinality |
| `OTEL_METRICS_INCLUDE_VERSION` | false | Enable for version tracking |
| `OTEL_METRICS_INCLUDE_ACCOUNT_UUID` | true | Disable if no per-user needed |

## Privacy

- Telemetry is opt-in (`CLAUDE_CODE_ENABLE_TELEMETRY=1`)
- Prompt content NOT logged by default (only length)
- Enable with `OTEL_LOG_USER_PROMPTS=1` (caution: may contain sensitive data)
- Tool inputs not logged by default; enable with `OTEL_LOG_TOOL_DETAILS=1`
