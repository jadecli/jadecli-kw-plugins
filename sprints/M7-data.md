# Sprint M7: Data — "We have real data"

**Branch**: `sprint/M7-data`
**Tag on merge**: `v0.7.0`
**Depends on**: M1 (v0.1.0)
**Brand**: jadecli runs on production data

## Task Queue (pgqueuer pattern — serial data pipeline)

```sql
INSERT INTO task_queue (sprint, task_id, task_type, title, depends_on, status)
VALUES
  ('M7', 'CD-M7-01', 'code',     'Deploy Kimball dimension tables',        NULL,       'queued'),
  ('M7', 'CD-M7-02', 'code',     'Deploy Kimball fact tables',             'CD-M7-01', 'queued'),
  ('M7', 'CD-M7-03', 'code',     'Verify RLS on all tables',              'CD-M7-02', 'queued'),
  ('M7', 'CD-M7-04', 'code',     'Deploy Cube.js model definitions',      'CD-M7-02', 'queued'),
  ('M7', 'RS-M7-05', 'research', 'Validate real session data (>10)',       'CD-M7-03', 'queued'),
  ('M7', 'RS-M7-06', 'research', 'Validate real tool_result data (>100)',  'CD-M7-03', 'queued'),
  ('M7', 'RS-M7-07', 'research', 'Validate real token_usage data',        'CD-M7-03', 'queued'),
  ('M7', 'CD-M7-08', 'code',     'Cube.js pre-aggregations refresh',      'CD-M7-04', 'queued'),
  ('M7', 'CD-M7-09', 'code',     'Pipeline metadata headers on all DDL',  'CD-M7-02', 'queued'),
  ('M7', 'CD-M7-10', 'code',     'M7 milestone tests pass',              'ALL',       'queued');
```

## DSPy Signatures

```python
class DeployKimballTable(dspy.Signature):
    """Apply a single Kimball DDL file to Supabase. 1 table per file."""
    sql_file: str = dspy.InputField(desc="Path to .sql file")
    table_name: str = dspy.OutputField()
    grain: str = dspy.OutputField(desc="Extracted from -- GRAIN: header")
    load_type: str = dspy.OutputField(desc="append-only | SCD1 | SCD2 | full-refresh")
    rls_enabled: bool = dspy.OutputField()

class ValidateRealData(dspy.Signature):
    """Query Supabase to verify real OTel data exists (not mock)."""
    event_name: str = dspy.InputField()
    min_count: int = dspy.InputField()
    actual_count: int = dspy.OutputField()
    passed: bool = dspy.OutputField()
    sample_record: dict = dspy.OutputField(desc="Most recent record for inspection")
```

## Tasks

### CD-M7-01: Deploy dimension tables
**Owner**: data-engineering + analytics-engineering
**Action**: Apply `schema/dimensions/dim_agent.sql`, `dim_tool.sql`, `dim_model.sql`, `dim_date.sql`
**PASS**: `psql $SUPABASE_DB_URL -c "SELECT COUNT(*) FROM information_schema.tables WHERE table_name LIKE 'dim_%'" | grep -q 4`

### CD-M7-02: Deploy fact tables
**Depends on**: CD-M7-01 (dimensions must exist for FK references)
**Action**: Apply `schema/facts/fct_tool_calls.sql`, `fct_sessions.sql`, `fct_token_usage.sql`
**PASS**: 3 fact tables exist

### CD-M7-03: RLS on all tables
**PASS**: `psql $SUPABASE_DB_URL -c "SELECT tablename, rowsecurity FROM pg_tables WHERE schemaname='public'" | grep -v " t$" | wc -l | xargs test 1 -eq` (only header line, all tables have RLS)

### CD-M7-04: Cube.js models
**PASS**: `ls cube/model/*.yml | wc -l | xargs test 4 -le`

### RS-M7-05: Real sessions >10
**Type**: `research:` | **Owner**: analytics-engineering
**PASS**: `psql $SUPABASE_DB_URL -t -c "SELECT COUNT(*) FROM otel_events WHERE event_name='claude_code.session.stop'" | xargs test 10 -lt`

### RS-M7-06: Real tool_results >100
**PASS**: Same pattern, `claude_code.tool_result`, threshold 100

### RS-M7-07: Real token_usage
**PASS**: Same pattern, `claude_code.token_usage`, threshold 10

### CD-M7-08: Pre-aggregations refresh
**PASS**: Cube.js `daily_by_role` pre-agg has data within last hour

### CD-M7-09: Pipeline metadata headers
**PASS**: `for f in schema/**/*.sql; do head -1 "$f" | grep -q "^-- DESCRIPTION:" || exit 1; done`

### CD-M7-10: M7 tests
**PASS**: `npx vitest run tests/milestones/m7-data.test.ts` exits 0

## Sprint Gate

All 10 green → merge → tag `v0.7.0` → M8, M10 unblock.
