# Sprint M3: Plugins — "We have knowledge"

**Branch**: `sprint/M3-plugins`
**Tag on merge**: `v0.3.0`
**Depends on**: M1 (v0.1.0)
**Brand**: jadecli has 8 expert plugins

## Task Queue

```sql
INSERT INTO task_queue (sprint, task_id, task_type, title, depends_on, status)
VALUES
  ('M3', 'CD-M3-01', 'code',     'Validate marketplace.json schema',       NULL,       'queued'),
  ('M3', 'CD-M3-02', 'code',     'Install all 8 plugins at project scope', 'CD-M3-01', 'queued'),
  ('M3', 'RS-M3-03', 'research', 'Verify every SKILL.md has frontmatter',  NULL,       'queued'),
  ('M3', 'CD-M3-04', 'code',     'Smoke test each skill invocation',       'CD-M3-02', 'queued'),
  ('M3', 'CD-M3-05', 'code',     'Set semver version on all plugin.json',  NULL,       'queued'),
  ('M3', 'CW-M3-06', 'cowork',   'Invoke skills from Cowork desktop',      'CD-M3-02', 'queued'),
  ('M3', 'RS-M3-07', 'research', 'Map skill dependencies across plugins',  'CD-M3-04', 'queued'),
  ('M3', 'CD-M3-08', 'code',     'Add input_examples to top 5 skills',     'RS-M3-07', 'queued'),
  ('M3', 'CD-M3-09', 'code',     'Enable ToolSearch for deferred skills',  'CD-M3-08', 'queued'),
  ('M3', 'CD-M3-10', 'code',     'M3 milestone tests pass',               'ALL',       'queued');
```

## DSPy Signatures

```python
class InstallPlugin(dspy.Signature):
    """Install a plugin from jadecli-kw-plugins marketplace."""
    plugin_name: str = dspy.InputField()
    marketplace: str = dspy.InputField(default="jadecli-kw-plugins")
    scope: str = dspy.InputField(default="project")
    installed: bool = dspy.OutputField()
    version: str = dspy.OutputField()
    skill_count: int = dspy.OutputField()

class SmokeTestSkill(dspy.Signature):
    """Invoke a skill and verify it returns without error."""
    skill_fqn: str = dspy.InputField(desc="plugin-name:skill-name")
    arguments: str = dspy.InputField(default="")
    success: bool = dspy.OutputField()
    error_message: str = dspy.OutputField(default=None)
```

## Tasks

### CD-M3-01: Validate marketplace.json
**PASS**: `jq -e '.plugins | length >= 8' .claude-plugin/marketplace.json`

### CD-M3-02: Install all 8 plugins
**PASS**: `claude plugin list 2>&1 | grep -c "enabled" | xargs test 8 -le`

### RS-M3-03: Verify SKILL.md frontmatter
**PASS**: `find . -name SKILL.md -exec head -3 {} \; | grep -c "^name:" | xargs test $(find . -name SKILL.md | wc -l) -eq`

### CD-M3-04: Smoke test skills
**PASS**: Each skill invoked with `--help` or minimal args returns without crash

### CW-M3-06: Skills from Cowork
**Type**: `cowork:` | **Owner**: software-dev-manager
**PASS**: Cowork desktop invokes `data-engineering:ast-index` and returns output

### RS-M3-07: Map skill dependencies
**Type**: `research:` | **Owner**: claude-architect
**PASS**: `docs/skill-dependency-map.md` documents which skills block others (e.g., analytics-eng blocked until data-eng creates tables)

### CD-M3-08: Add input_examples (Advanced Tool Use)
**PASS**: Top 5 most-used skills have `input_examples` in their definitions

### CD-M3-09: ToolSearch for deferred skills
**PASS**: `ENABLE_TOOL_SEARCH=auto:5` set in managed settings; deferred tools load on-demand

### CD-M3-10: M3 milestone tests
**PASS**: `npx vitest run tests/milestones/m3-plugins.test.ts` exits 0

## Sprint Gate

All 10 green → merge → tag `v0.3.0` → M4 unblocks.
