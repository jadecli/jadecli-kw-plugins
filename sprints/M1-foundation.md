# Sprint M1: Foundation — "We have telemetry"

**Branch**: `sprint/M1-foundation`
**Tag on merge**: `v0.1.0`
**Brand**: jadecli measures everything

## Task Queue (Supabase pgqueuer pattern)

Tasks execute in sequence. Each task's PASS condition must be green before the next dequeues. Dependencies enforced via `execute_after` — downstream tasks blocked until upstream completes with `status: successful`.

```sql
-- pgqueuer job table on Supabase
INSERT INTO task_queue (sprint, task_id, task_type, title, depends_on, status)
VALUES
  ('M1', 'CD-M1-01', 'code',     'Create otel_events schema',           NULL,        'queued'),
  ('M1', 'CD-M1-02', 'code',     'Create otel_metrics schema',          'CD-M1-01',  'queued'),
  ('M1', 'CD-M1-03', 'code',     'Deploy managed OTel settings',        'CD-M1-02',  'queued'),
  ('M1', 'RS-M1-04', 'research', 'Validate OTel collector routing',     'CD-M1-03',  'queued'),
  ('M1', 'CD-M1-05', 'code',     'Remove all ANTHROPIC_API_KEY refs',   NULL,        'queued'),
  ('M1', 'CD-M1-06', 'code',     'AST index builds clean',             NULL,        'queued'),
  ('M1', 'CW-M1-07', 'cowork',   'Verify telemetry in Cowork session', 'CD-M1-03',  'queued'),
  ('M1', 'CD-M1-08', 'code',     'CLAUDE.md hierarchy correct',         NULL,        'queued'),
  ('M1', 'CD-M1-09', 'code',     '.gitignore complete',                NULL,        'queued'),
  ('M1', 'CD-M1-10', 'code',     'M1 milestone tests pass',            'ALL',       'queued');
```

## DSPy Signatures (structured I/O per task)

```python
# Each task is a DSPy Signature — typed input → typed output
# No prompting. Programming.

class CreateOtelSchema(dspy.Signature):
    """Deploy otel_events + otel_metrics tables to Supabase."""
    supabase_url: str = dspy.InputField(desc="SUPABASE_URL env var")
    migration_files: list[str] = dspy.InputField(desc="SQL files to apply")
    tables_created: list[str] = dspy.OutputField(desc="Tables successfully created")
    rls_enabled: bool = dspy.OutputField(desc="RLS enabled on all tables")

class ValidateOtelPipeline(dspy.Signature):
    """Verify OTel events flow from Claude Code → Supabase."""
    collector_endpoint: str = dspy.InputField()
    event_count: int = dspy.OutputField(desc="Events in last hour, must be > 0")
    latency_ms: float = dspy.OutputField(desc="P95 ingestion latency")
```

## Tasks

### CD-M1-01: Create otel_events schema
**Type**: `code:`
**Owner**: data-engineering
**Input**: `schema/001_otel_events.sql`
**Action**: Apply migration to Supabase
```bash
psql $SUPABASE_DB_URL -f schema/001_otel_events.sql
```
**PASS**: `psql $SUPABASE_DB_URL -c "SELECT COUNT(*) FROM information_schema.tables WHERE table_name = 'otel_events'" | grep -q 1`

### CD-M1-02: Create otel_metrics schema
**Type**: `code:`
**Owner**: data-engineering
**Depends on**: CD-M1-01
**Action**: Apply metrics table migration
**PASS**: `psql $SUPABASE_DB_URL -c "SELECT COUNT(*) FROM information_schema.tables WHERE table_name = 'otel_metrics'" | grep -q 1`

### CD-M1-03: Deploy managed OTel settings
**Type**: `code:`
**Owner**: software-dev-manager
**Depends on**: CD-M1-02
**Action**: Write managed settings JSON with OTel env vars
**PASS**: `cat managed-settings.json | jq -e '.env.CLAUDE_CODE_ENABLE_TELEMETRY == "1"'`

### RS-M1-04: Validate OTel collector routing
**Type**: `research:`
**Owner**: integration-engineering
**Depends on**: CD-M1-03
**Action**: Verify collector forwards events to Supabase endpoint
**PASS**: `psql $SUPABASE_DB_URL -t -c "SELECT COUNT(*) FROM otel_events WHERE recorded_at > now() - INTERVAL '1 hour'" | xargs test 0 -lt`

### CD-M1-05: Remove all ANTHROPIC_API_KEY refs
**Type**: `code:`
**Owner**: claude-qae
**Action**: `grep -r ANTHROPIC_API_KEY .github/ src/ scripts/` → fix all hits
**PASS**: `grep -r ANTHROPIC_API_KEY .github/ src/ scripts/ | wc -l | xargs test 0 -eq`

### CD-M1-06: AST index builds clean
**Type**: `code:`
**Owner**: data-engineering
**Action**: `source .venv/bin/activate && python scripts/build-ast-index.py`
**PASS**: `test -s ast-index/L2-symbols.json && jq length ast-index/L2-symbols.json | xargs test 0 -lt`

### CW-M1-07: Verify telemetry in Cowork session
**Type**: `cowork:`
**Owner**: software-dev-manager
**Depends on**: CD-M1-03
**Action**: Open Cowork session, run a task, check Supabase for the session event
**PASS**: Session event with matching session_id appears in `otel_events` within 5 minutes

### CD-M1-08: CLAUDE.md hierarchy correct
**Type**: `code:`
**Owner**: claude-architect
**Action**: Verify root CLAUDE.md loads, .claude/rules/ scoped correctly
**PASS**: `test -f CLAUDE.md && test -d .claude/rules/`

### CD-M1-09: .gitignore complete
**Type**: `code:`
**Owner**: claude-qae
**Action**: Ensure .venv, ast-index, node_modules excluded
**PASS**: `grep -q '.venv' .gitignore && grep -q 'ast-index' .gitignore && grep -q 'node_modules' .gitignore`

### CD-M1-10: M1 milestone tests pass
**Type**: `code:`
**Owner**: claude-qae
**Depends on**: ALL above
**Action**: Run full M1 test suite
**PASS**: `npx vitest run tests/milestones/m1-foundation.test.ts` exits 0

## Sprint Gate

**All 10 PASS conditions green** → merge `sprint/M1-foundation` → tag `v0.1.0` → M2 unblocks.
