# Sprint M10: Roadmap — "We have a roadmap"

**Branch**: `sprint/M10-roadmap`
**Tag on merge**: `v0.10.0` → `v1.0.0` (production)
**Depends on**: M5 (v0.5.0), M7 (v0.7.0)
**Brand**: jadecli plans and measures ROI

## Task Queue (pgqueuer — final pipeline)

```sql
INSERT INTO task_queue (sprint, task_id, task_type, title, depends_on, status)
VALUES
  ('M10', 'CD-M10-01', 'code',     'Connect Linear MCP with project data',  NULL,       'queued'),
  ('M10', 'RS-M10-02', 'research', 'Verify Linear has product roadmap',     'CD-M10-01','queued'),
  ('M10', 'CD-M10-03', 'code',     'Current sprint has ≥5 issues',          'CD-M10-01','queued'),
  ('M10', 'CD-M10-04', 'code',     'WBR cron fires successfully',           NULL,       'queued'),
  ('M10', 'RS-M10-05', 'research', 'WBR report has real metrics',           'CD-M10-04','queued'),
  ('M10', 'CD-M10-06', 'code',     'Per-role KW ROI in dashboard',          NULL,       'queued'),
  ('M10', 'RS-M10-07', 'research', 'Quarterly OKRs defined in Linear',      'CD-M10-01','queued'),
  ('M10', 'CW-M10-08', 'cowork',   'Review WBR from Cowork/mobile',         'RS-M10-05','queued'),
  ('M10', 'CD-M10-09', 'code',     'pgqueuer task_queue table deployed',    NULL,       'queued'),
  ('M10', 'CD-M10-10', 'code',     'ALL milestone tests pass (M1-M10)',    'ALL',       'queued');
```

## DSPy Signatures

```python
class GenerateWBR(dspy.Signature):
    """Weekly Business Review from OTel + Linear data."""
    week_start: str = dspy.InputField(desc="ISO date of Monday")
    otel_summary: dict = dspy.InputField(desc="TelemetrySummary from Supabase")
    linear_sprint: dict = dspy.InputField(desc="Current sprint issues + status")
    wbr_report: str = dspy.OutputField(desc="Markdown report with per-role metrics")
    headlines: list[str] = dspy.OutputField(desc="3-5 key takeaways")
    actions: list[str] = dspy.OutputField(desc="Next action items")

class ValidateRoadmap(dspy.Signature):
    """Verify Linear has a product roadmap with milestones and OKRs."""
    linear_project: str = dspy.InputField()
    has_roadmap: bool = dspy.OutputField()
    milestone_count: int = dspy.OutputField()
    okr_count: int = dspy.OutputField()
    current_sprint_issues: int = dspy.OutputField()

class DeployTaskQueue(dspy.Signature):
    """Deploy pgqueuer-style task_queue table to Supabase for sprint orchestration."""
    schema_file: str = dspy.InputField()
    table_created: bool = dspy.OutputField()
    listen_notify_enabled: bool = dspy.OutputField()
    indexes_created: list[str] = dspy.OutputField()
```

## Tasks

### CD-M10-01: Linear MCP connected
**Owner**: software-dev-manager
**PASS**: `linear:list_issues` MCP call returns jadecli project issues

### RS-M10-02: Product roadmap exists
**PASS**: Linear project has milestones with due dates

### CD-M10-03: Sprint has issues
**PASS**: Current sprint has ≥5 issues with status

### CD-M10-04: WBR cron fires
**PASS**: `gh run list --workflow=wbr-cron.yml --limit 1 --json conclusion --jq '.[0].conclusion'` returns "success"

### RS-M10-05: WBR has real metrics
**PASS**: WBR report contains non-zero PRs, cost, and session data for current week

### CD-M10-06: Per-role ROI in dashboard
**PASS**: Dashboard shows cost/lines/PRs broken down by role from `OTEL_RESOURCE_ATTRIBUTES`

### RS-M10-07: OKRs defined
**PASS**: Linear goals/OKRs exist for current quarter with measurable key results

### CW-M10-08: Review WBR from mobile
**Type**: `cowork:` | **Owner**: software-dev-manager
**PASS**: WBR report viewable and actionable from Cowork mobile

### CD-M10-09: pgqueuer task_queue deployed
**Owner**: data-engineering
**Action**: Deploy task queue infrastructure to Supabase

```sql
-- schema/infra/task_queue.sql
-- DESCRIPTION: pgqueuer-pattern task queue for sprint orchestration
-- GRAIN: one row per task
-- LOAD_TYPE: append + status updates
-- SOURCE: architect milestone planning
-- SCHEDULE: on-demand (event-driven via LISTEN/NOTIFY)

CREATE TABLE IF NOT EXISTS task_queue (
  id BIGSERIAL PRIMARY KEY,
  sprint TEXT NOT NULL,
  task_id TEXT NOT NULL UNIQUE,
  task_type TEXT NOT NULL CHECK (task_type IN ('code', 'cowork', 'research')),
  title TEXT NOT NULL,
  owner TEXT,
  depends_on TEXT,  -- task_id or 'ALL'
  status TEXT NOT NULL DEFAULT 'queued'
    CHECK (status IN ('queued', 'running', 'successful', 'failed', 'blocked')),
  started_at TIMESTAMPTZ,
  completed_at TIMESTAMPTZ,
  result JSONB DEFAULT '{}',
  created_at TIMESTAMPTZ DEFAULT now()
);

CREATE INDEX idx_task_queue_sprint ON task_queue(sprint);
CREATE INDEX idx_task_queue_status ON task_queue(status);
CREATE INDEX idx_task_queue_depends ON task_queue(depends_on);

-- LISTEN/NOTIFY for real-time task completion events
CREATE OR REPLACE FUNCTION notify_task_complete() RETURNS trigger AS $$
BEGIN
  IF NEW.status IN ('successful', 'failed') THEN
    PERFORM pg_notify('task_complete', json_build_object(
      'task_id', NEW.task_id,
      'sprint', NEW.sprint,
      'status', NEW.status
    )::text);
  END IF;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER task_complete_trigger
  AFTER UPDATE OF status ON task_queue
  FOR EACH ROW EXECUTE FUNCTION notify_task_complete();

ALTER TABLE task_queue ENABLE ROW LEVEL SECURITY;
```

**PASS**: `psql $SUPABASE_DB_URL -c "SELECT COUNT(*) FROM information_schema.tables WHERE table_name='task_queue'" | grep -q 1` AND `psql $SUPABASE_DB_URL -c "SELECT COUNT(*) FROM information_schema.triggers WHERE trigger_name='task_complete_trigger'" | grep -q 1`

### CD-M10-10: ALL milestone tests pass (M1-M10)
**Owner**: claude-qae
**Action**: Run the full milestone test suite
**PASS**: `npx vitest run tests/milestones/` exits 0 with all 100 conditions green

## Sprint Gate

All 10 green → merge → tag `v0.10.0` → tag `v1.0.0` (production release)

## v1.0.0 Checklist

When M10 merges, the jadecli ecosystem is complete:

- [x] M1: OTel → Supabase ✓
- [x] M2: All MCPs connected ✓
- [x] M3: 8 KW plugins active ✓
- [x] M4: Agent teams pass ✓
- [x] M5: Slack + Channels + RC + Dispatch ✓
- [x] M6: CI green, review posting ✓
- [x] M7: Real data in Supabase ✓
- [x] M8: jadecli.com live with dashboards ✓
- [x] M9: iOS mobile E2E ✓
- [x] M10: Linear roadmap + WBR + pgqueuer task queue ✓

**100 deterministic conditions. All green. Ship it.**
