---
name: supabase-cube
description: Setup and manage Supabase PostgreSQL tables with Cube.js semantic layer definitions. Creates tables, migrations, and matching Cube models.
argument-hint: "<init|model|migrate|query> [entity-name]"
---

# /cube-model - Supabase + Cube.js Semantic Layer

## Usage

```
/cube-model init                          # scaffold project
/cube-model model agents                  # create model + table
/cube-model migrate                       # apply migrations
/cube-model query "total agents by role"  # test semantic query
```

## Project Structure

```
cube/
├── model/                    # Cube.js semantic models
│   ├── agents.yml
│   ├── tasks.yml
│   ├── crawl_runs.yml
│   └── token_usage.yml
├── schema/                   # Supabase migrations
│   ├── 001_agents.sql
│   ├── 002_tasks.sql
│   ├── 003_crawl_runs.sql
│   └── 004_token_usage.sql
├── pre-aggregations/
│   └── daily_rollups.yml
└── cube.js                   # Config (Supabase as data source)
```

## Core Models

| Model | Measures | Dimensions |
|-------|----------|------------|
| `agents` | count, total_tokens, total_cost | role, model, domain, status |
| `tasks` | count, avg_duration, completion_rate | agent_id, status, priority |
| `crawl_runs` | count, items_scraped, error_rate | spider, source |
| `token_usage` | total_tokens, total_cost | operation, model, date |

## Example: Create "agents" Model

### Supabase Table

```sql
CREATE TABLE IF NOT EXISTS agents (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  role TEXT NOT NULL CHECK (role IN ('researcher','engineer','reviewer','operator','guide')),
  model TEXT NOT NULL DEFAULT 'claude-sonnet-4-6',
  domain TEXT,
  budget_tokens INTEGER DEFAULT 100000,
  budget_usd NUMERIC(10,4) DEFAULT 1.00,
  status TEXT NOT NULL DEFAULT 'idle',
  tokens_used INTEGER DEFAULT 0,
  cost_usd NUMERIC(10,4) DEFAULT 0,
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
);
ALTER TABLE agents ENABLE ROW LEVEL SECURITY;
CREATE INDEX idx_agents_role ON agents(role);
CREATE INDEX idx_agents_status ON agents(status);
```

### Cube.js Model

```yaml
cubes:
  - name: agents
    sql_table: agents
    data_source: supabase
    measures:
      - name: count
        type: count
      - name: total_tokens_used
        type: sum
        sql: tokens_used
      - name: total_cost
        type: sum
        sql: cost_usd
    dimensions:
      - name: id
        sql: id
        type: string
        primary_key: true
      - name: role
        sql: role
        type: string
      - name: status
        sql: status
        type: string
      - name: created_at
        sql: created_at
        type: time
    pre_aggregations:
      - name: daily_by_role
        measures: [count, total_tokens_used, total_cost]
        dimensions: [role]
        time_dimension: created_at
        granularity: day
```

## Cube.js Config

```javascript
module.exports = {
  dbType: 'postgres',
  driverFactory: () => ({
    type: 'postgres',
    database: process.env.SUPABASE_DB_NAME || 'postgres',
    host: process.env.SUPABASE_DB_HOST,
    port: 5432,
    user: 'postgres',
    password: process.env.SUPABASE_DB_PASSWORD,
    ssl: { rejectUnauthorized: false },
  }),
};
```

## Migrate

```bash
supabase db push --db-url $SUPABASE_DB_URL
```

## Safety

- Always enable RLS on Supabase tables
- Never expose `service_role` key in client code
- Migrations are append-only — never modify existing files
- Pre-aggregations reduce query cost — use for dashboards
