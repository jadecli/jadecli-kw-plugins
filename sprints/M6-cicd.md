# Sprint M6: CI/CD — "We ship safely"

**Branch**: `sprint/M6-cicd`
**Tag on merge**: `v0.6.0`
**Depends on**: M2 (v0.2.0)
**Brand**: jadecli's CI catches everything

## Task Queue

```sql
INSERT INTO task_queue (sprint, task_id, task_type, title, depends_on, status)
VALUES
  ('M6', 'CD-M6-01', 'code',     'Deploy claude.yml workflow',             NULL,       'queued'),
  ('M6', 'CD-M6-02', 'code',     'Deploy claude-code-review.yml',          NULL,       'queued'),
  ('M6', 'CD-M6-03', 'code',     'All workflows use OAUTH_TOKEN',          NULL,       'queued'),
  ('M6', 'RS-M6-04', 'research', 'Verify branch protection rules',         NULL,       'queued'),
  ('M6', 'CD-M6-05', 'code',     'Enable squash-only merge',              'RS-M6-04', 'queued'),
  ('M6', 'CD-M6-06', 'code',     'commitlint hook rejects bad titles',    NULL,       'queued'),
  ('M6', 'CD-M6-07', 'code',     'CI pipeline green on main',             'CD-M6-01', 'queued'),
  ('M6', 'CW-M6-08', 'cowork',   '@claude mention triggers workflow',      'CD-M6-01', 'queued'),
  ('M6', 'CW-M6-09', 'cowork',   'Code review posts inline comments',     'CD-M6-02', 'queued'),
  ('M6', 'CD-M6-10', 'code',     'M6 milestone tests pass',              'ALL',       'queued');
```

## DSPy Signatures

```python
class ValidateCIPipeline(dspy.Signature):
    """Verify CI pipeline is green and all gates are configured."""
    repo: str = dspy.InputField()
    branch: str = dspy.InputField(default="main")
    latest_run_status: str = dspy.OutputField(desc="success | failure")
    required_checks: list[str] = dspy.OutputField()
    branch_protection_enabled: bool = dspy.OutputField()
    squash_only: bool = dspy.OutputField()
```

## Tasks

### CD-M6-01: Deploy claude.yml
**PASS**: `gh api repos/jadecli/jadebot/contents/.github/workflows/claude.yml --jq '.name'` returns file

### CD-M6-02: Deploy claude-code-review.yml
**PASS**: Same check for review workflow

### CD-M6-03: All workflows use OAUTH_TOKEN
**PASS**: `grep -r anthropic_api_key .github/workflows/ | wc -l | xargs test 0 -eq` AND `grep -r claude_code_oauth_token .github/workflows/ | wc -l | xargs test 0 -lt`

### RS-M6-04: Branch protection
**PASS**: `gh api repos/jadecli/jadebot/branches/main/protection --jq '.enforce_admins.enabled'` returns true

### CD-M6-05: Squash-only
**PASS**: `gh api repos/jadecli/jadebot --jq '.allow_merge_commit'` returns false

### CD-M6-06: commitlint rejects bad titles
**PASS**: `echo "bad title" | npx commitlint` exits non-zero

### CD-M6-07: CI green on main
**PASS**: `gh run list --branch main --limit 1 --json conclusion --jq '.[0].conclusion'` returns "success"

### CW-M6-08: @claude triggers workflow
**Type**: `cowork:` | **Owner**: integration-engineering
**PASS**: PR comment `@claude review this` → workflow run appears within 5 minutes

### CW-M6-09: Review posts comments
**Type**: `cowork:` | **Owner**: claude-qae
**PASS**: PR has `Claude Code Review` check run with ≥1 inline comment

### CD-M6-10: M6 tests
**PASS**: `npx vitest run tests/milestones/m6-cicd.test.ts` exits 0

## Sprint Gate

All 10 green → merge → tag `v0.6.0`.
