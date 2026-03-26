---
name: kimball-modeling
description: Kimball dimensional modeling methodology with semantic layer declarations, 1-table-per-file DDL, load types, pipeline metadata, and version-controlled business logic. Based on The Data Warehouse Toolkit.
argument-hint: "<grain|dimension|fact|pipeline|semantic> [entity-name]"
---

# Kimball Dimensional Modeling

Dimensional modeling following Kimball Group methodology (The Data Warehouse Toolkit), enhanced with:
- **Semantic layer declarations** per table (Cube.js YAML)
- **1 table per file** — DDL and business logic version controlled
- **Load type annotations** — SCD type, refresh strategy, grain
- **Pipeline metadata** — source, schedule, dependencies

## The 4-Step Process

### 1. Select the Business Process

Identify the operational process being measured. One fact table per process.

```
Business process: Claude Code agent sessions
Grain: one row per tool call per session
```

### 2. Declare the Grain

The grain is the most atomic level of data. Declare it explicitly in every fact table file.

```sql
-- schema/facts/fct_tool_calls.sql
-- GRAIN: one row per tool_result event per session
-- LOAD_TYPE: append-only (immutable events)
-- SOURCE: otel_events WHERE event_name = 'claude_code.tool_result'
-- SCHEDULE: continuous (event-driven)
```

### 3. Identify the Dimensions

Dimensions describe the "who, what, where, when, why, how" context.

### 4. Identify the Facts

Facts are the numeric measurements of the business process.

## File Convention — 1 Table Per File

```
schema/
├── dimensions/
│   ├── dim_agent.sql          # Agent dimension (type, model, domain)
│   ├── dim_tool.sql           # Tool dimension (name, category)
│   ├── dim_model.sql          # LLM model dimension (name, pricing)
│   └── dim_date.sql           # Date dimension (standard Kimball)
├── facts/
│   ├── fct_tool_calls.sql     # Tool call fact (grain: 1 row per tool call)
│   ├── fct_sessions.sql       # Session fact (grain: 1 row per session)
│   └── fct_token_usage.sql    # Token usage fact (grain: 1 row per API call)
├── staging/
│   ├── stg_otel_events.sql    # Cleaned otel_events
│   └── stg_otel_metrics.sql   # Cleaned otel_metrics
└── semantic/
    ├── dim_agent.yml           # Cube.js semantic model
    ├── dim_tool.yml
    ├── fct_tool_calls.yml
    ├── fct_sessions.yml
    └── fct_token_usage.yml
```

## DDL Template — Dimension

```sql
-- schema/dimensions/dim_agent.sql
-- DESCRIPTION: Agent dimension — agent types, models, and domains
-- LOAD_TYPE: SCD Type 1 (overwrite on change)
-- SOURCE: otel_events attributes->>'agent_type'
-- SCHEDULE: daily at 03:00 UTC
-- DEPENDS_ON: stg_otel_events

CREATE TABLE IF NOT EXISTS dim_agent (
  agent_key SERIAL PRIMARY KEY,
  agent_type TEXT NOT NULL UNIQUE,
  default_model TEXT,
  domain TEXT,
  -- SCD metadata
  effective_from TIMESTAMPTZ DEFAULT now(),
  is_current BOOLEAN DEFAULT true,
  -- Audit
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
);

-- Business logic: deduplicate from staging
INSERT INTO dim_agent (agent_type, default_model, domain)
SELECT DISTINCT
  attributes->>'agent_type',
  attributes->>'model',
  attributes->>'domain'
FROM stg_otel_events
WHERE event_name = 'claude_code.session.stop'
ON CONFLICT (agent_type) DO UPDATE SET
  default_model = EXCLUDED.default_model,
  updated_at = now();
```

## DDL Template — Fact

```sql
-- schema/facts/fct_tool_calls.sql
-- DESCRIPTION: Tool call fact table — one row per tool_result event
-- GRAIN: one row per tool call per session
-- LOAD_TYPE: append-only (immutable event stream)
-- SOURCE: otel_events WHERE event_name = 'claude_code.tool_result'
-- SCHEDULE: continuous / micro-batch every 5 min
-- DEPENDS_ON: stg_otel_events, dim_tool, dim_agent

CREATE TABLE IF NOT EXISTS fct_tool_calls (
  tool_call_key BIGSERIAL PRIMARY KEY,
  -- Dimension FKs
  session_id TEXT NOT NULL,
  tool_key INT REFERENCES dim_tool(tool_key),
  agent_key INT REFERENCES dim_agent(agent_key),
  date_key INT REFERENCES dim_date(date_key),
  -- Facts (measures)
  success BOOLEAN NOT NULL,
  duration_ms FLOAT,
  -- Degenerate dimensions
  event_id BIGINT UNIQUE,  -- otel_events.id for idempotency
  -- Audit
  recorded_at TIMESTAMPTZ NOT NULL,
  loaded_at TIMESTAMPTZ DEFAULT now()
);

CREATE INDEX idx_fct_tool_calls_session ON fct_tool_calls(session_id);
CREATE INDEX idx_fct_tool_calls_recorded ON fct_tool_calls(recorded_at);
```

## Semantic Layer — Cube.js YAML

```yaml
# semantic/fct_tool_calls.yml
cubes:
  - name: tool_calls
    sql_table: fct_tool_calls
    data_source: supabase

    joins:
      - name: tool
        sql: "{CUBE}.tool_key = {dim_tool}.tool_key"
        relationship: many_to_one
      - name: agent
        sql: "{CUBE}.agent_key = {dim_agent}.agent_key"
        relationship: many_to_one

    measures:
      - name: count
        type: count
      - name: success_count
        type: count
        filters:
          - sql: "{CUBE}.success = true"
      - name: error_count
        type: count
        filters:
          - sql: "{CUBE}.success = false"
      - name: success_rate
        type: number
        sql: "{success_count}::float / NULLIF({count}, 0)"
      - name: avg_duration_ms
        type: avg
        sql: duration_ms
      - name: p95_duration_ms
        type: number
        sql: "PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY {CUBE}.duration_ms)"

    dimensions:
      - name: session_id
        sql: session_id
        type: string
      - name: success
        sql: success
        type: boolean
      - name: recorded_at
        sql: recorded_at
        type: time

    pre_aggregations:
      - name: daily_by_tool
        measures: [count, success_count, error_count, avg_duration_ms]
        dimensions: [tool.tool_name]
        time_dimension: recorded_at
        granularity: day
        refresh_key:
          every: 1 hour
```

## Load Types

| Load Type | When | Example |
|-----------|------|---------|
| **Append-only** | Immutable events | fct_tool_calls, fct_token_usage |
| **SCD Type 1** | Overwrite on change | dim_agent, dim_tool |
| **SCD Type 2** | Track history | dim_model (pricing changes) |
| **Full refresh** | Small reference data | dim_date |
| **Incremental** | Large + mutable | fct_sessions (backfill corrections) |

## Pipeline Metadata Header

Every SQL file starts with a metadata header:

```sql
-- DESCRIPTION: <what this table represents>
-- GRAIN: <one row per ...>
-- LOAD_TYPE: <append-only | SCD1 | SCD2 | full-refresh | incremental>
-- SOURCE: <source table or system>
-- SCHEDULE: <cron or frequency>
-- DEPENDS_ON: <upstream tables>
-- OWNER: <team or person>
-- TAGS: <comma-separated tags>
```

## Kimball Best Practices

1. **Grain first** — always declare grain before adding columns
2. **Conformed dimensions** — shared across fact tables (dim_date, dim_agent)
3. **Surrogate keys** — use SERIAL/BIGSERIAL, never business keys as PKs
4. **Degenerate dimensions** — business keys stored directly in fact (session_id, event_id)
5. **Slowly changing dimensions** — annotate SCD type per dimension
6. **Bus matrix** — map business processes to dimensions
7. **Idempotent loads** — use event_id for deduplication on re-runs
8. **Version control** — 1 table per file, business logic in SQL, reviewed via PR

## Reference

- Kimball, R. & Ross, M. (2013). *The Data Warehouse Toolkit: The Definitive Guide to Dimensional Modeling* (3rd ed.). Wiley.
- https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/books/data-warehouse-dw-toolkit/
