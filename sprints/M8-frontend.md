# Sprint M8: Frontend — "We have a product"

**Branch**: `sprint/M8-frontend`
**Tag on merge**: `v0.8.0`
**Depends on**: M5 (v0.5.0), M7 (v0.7.0)
**Brand**: jadecli.com is live with dashboards

## Task Queue

```sql
INSERT INTO task_queue (sprint, task_id, task_type, title, depends_on, status)
VALUES
  ('M8', 'CD-M8-01', 'code',     'Netlify deploys jadecli.com (200)',      NULL,       'queued'),
  ('M8', 'CD-M8-02', 'code',     'Set SUPABASE_URL in Netlify env',       NULL,       'queued'),
  ('M8', 'CD-M8-03', 'code',     'Telemetry dashboard page renders',       'CD-M8-02', 'queued'),
  ('M8', 'RS-M8-04', 'research', 'Dashboard shows real data (not mock)',   'CD-M8-03', 'queued'),
  ('M8', 'CD-M8-05', 'code',     'Tool breakdown table populated',        'CD-M8-03', 'queued'),
  ('M8', 'CD-M8-06', 'code',     'Cost cards show USD values',            'CD-M8-03', 'queued'),
  ('M8', 'CD-M8-07', 'code',     'Vercel console app builds',             NULL,       'queued'),
  ('M8', 'CW-M8-08', 'cowork',   'Review dashboard UX in Cowork',         'CD-M8-03', 'queued'),
  ('M8', 'CD-M8-09', 'code',     'turbo-ignore skips bot branches',       NULL,       'queued'),
  ('M8', 'CD-M8-10', 'code',     'M8 milestone tests pass',              'ALL',       'queued');
```

## DSPy Signatures

```python
class ValidateDashboard(dspy.Signature):
    """Verify telemetry dashboard renders with real data."""
    dashboard_url: str = dspy.InputField()
    total_sessions: int = dspy.OutputField(desc="Must be > 0")
    tool_breakdown_count: int = dspy.OutputField(desc="Unique tools, must be > 0")
    cost_usd: float = dspy.OutputField(desc="Must be > 0")
    is_mock_data: bool = dspy.OutputField(desc="Must be false")
```

## Tasks

### CD-M8-01: jadecli.com returns 200
**PASS**: `curl -s -o /dev/null -w '%{http_code}' https://jadecli.com | grep -q 200`

### CD-M8-02: Supabase env in Netlify
**PASS**: `netlify env:get SUPABASE_URL` returns non-empty

### CD-M8-03: Dashboard page renders
**PASS**: `curl -s https://jadecli.com/telemetry | grep -q StatCard`

### RS-M8-04: Real data (not mock)
**PASS**: `fetchSummary().totalSessions > 0` from server-side query

### CD-M8-05: Tool breakdown
**PASS**: `fetchToolBreakdown().length > 0`

### CD-M8-06: Cost cards
**PASS**: `fetchCostSummary().totalCostUsd > 0`

### CD-M8-07: Vercel console builds
**PASS**: `cd apps/console && npx next build` exits 0

### CW-M8-08: Review dashboard in Cowork
**Type**: `cowork:` | **Owner**: frontend-engineering + software-dev-manager
**PASS**: Dashboard reviewed for readability, stat cards visible, no broken components

### CD-M8-09: turbo-ignore skips bots
**PASS**: `cat apps/console/vercel.json | jq -e '.ignoreCommand | contains("dependabot")'`

### CD-M8-10: M8 tests
**PASS**: `npx vitest run tests/milestones/m8-frontend.test.ts` exits 0

## Sprint Gate

All 10 green → merge → tag `v0.8.0` → M9 unblocks.
