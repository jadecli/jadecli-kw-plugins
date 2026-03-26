# Sprint M2: Tools & MCP — "We have tools"

**Branch**: `sprint/M2-tools-mcp`
**Tag on merge**: `v0.2.0`
**Depends on**: M1 (v0.1.0)
**Brand**: jadecli connects to anything

## Task Queue

```sql
INSERT INTO task_queue (sprint, task_id, task_type, title, depends_on, status)
VALUES
  ('M2', 'CD-M2-01', 'code',     'Configure Supabase MCP server',          NULL,       'queued'),
  ('M2', 'CD-M2-02', 'code',     'Configure GitHub MCP server',            NULL,       'queued'),
  ('M2', 'CD-M2-03', 'code',     'Configure Linear MCP server',            NULL,       'queued'),
  ('M2', 'CD-M2-04', 'code',     'Configure jade-console-mcp',             NULL,       'queued'),
  ('M2', 'RS-M2-05', 'research', 'Audit tool descriptions for overlap',    'CD-M2-01', 'queued'),
  ('M2', 'CD-M2-06', 'code',     'Implement structured MCP error responses','CD-M2-01','queued'),
  ('M2', 'CD-M2-07', 'code',     'Enforce ≤8 tools per agent definition',  NULL,       'queued'),
  ('M2', 'CW-M2-08', 'cowork',   'Test MCP tools from Cowork app',         'CD-M2-01', 'queued'),
  ('M2', 'RS-M2-09', 'research', 'Document tool_choice patterns per agent', 'RS-M2-05','queued'),
  ('M2', 'CD-M2-10', 'code',     'M2 milestone tests pass',               'ALL',       'queued');
```

## DSPy Signatures

```python
class ConfigureMCPServer(dspy.Signature):
    """Add an MCP server to .mcp.json with env var auth."""
    server_name: str = dspy.InputField()
    npm_package: str = dspy.InputField()
    env_vars: dict[str, str] = dspy.InputField(desc="e.g. {SUPABASE_URL: '${SUPABASE_URL}'}")
    connected: bool = dspy.OutputField()
    tools_discovered: list[str] = dspy.OutputField()

class AuditToolDescriptions(dspy.Signature):
    """Check all MCP tool descriptions for ambiguity and overlap."""
    tool_definitions: list[dict] = dspy.InputField()
    overlapping_pairs: list[tuple[str, str]] = dspy.OutputField()
    recommendations: list[str] = dspy.OutputField()
```

## Tasks

### CD-M2-01 through CD-M2-04: Configure MCP servers (parallel)
**Type**: `code:` | **Owner**: integration-engineering
**Action**: Add to `.mcp.json` with `${ENV_VAR}` auth
**PASS (each)**: MCP tool call returns non-error response

### RS-M2-05: Audit tool descriptions
**Type**: `research:` | **Owner**: claude-architect
**PASS**: No two tools share >80% description similarity (measured by jq string comparison)

### CD-M2-06: Structured MCP errors
**Type**: `code:` | **Owner**: claude-qae
**PASS**: Every MCP error response contains `isError`, `errorCategory`, `isRetryable`

### CD-M2-07: Tool count ≤8 per agent
**Type**: `code:` | **Owner**: claude-architect
**PASS**: `grep -c "allowedTools" .claude/agents/*.md | awk -F: '$2>8{found=1}END{exit found}'`

### CW-M2-08: Test MCPs from Cowork
**Type**: `cowork:` | **Owner**: software-dev-manager
**PASS**: Supabase query tool returns table list from Cowork desktop app

### RS-M2-09: Document tool_choice patterns
**Type**: `research:` | **Owner**: claude-architect
**PASS**: `docs/tool-choice-patterns.md` exists with per-agent guidance

### CD-M2-10: M2 milestone tests
**Type**: `code:` | **Owner**: claude-qae
**PASS**: `npx vitest run tests/milestones/m2-tools-mcp.test.ts` exits 0

## Sprint Gate

All 10 green → merge → tag `v0.2.0` → M5, M6 unblock.
