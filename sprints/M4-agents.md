# Sprint M4: Agents — "We have teams"

**Branch**: `sprint/M4-agents`
**Tag on merge**: `v0.4.0`
**Depends on**: M3 (v0.3.0)
**Brand**: jadecli reviews its own code

## Task Queue

```sql
INSERT INTO task_queue (sprint, task_id, task_type, title, depends_on, status)
VALUES
  ('M4', 'CD-M4-01', 'code',     'Staff review team tests pass',           NULL,       'queued'),
  ('M4', 'CD-M4-02', 'code',     'Skeptical codegen team tests pass',      NULL,       'queued'),
  ('M4', 'CD-M4-03', 'code',     'Agent factory TypeScript compiles',      NULL,       'queued'),
  ('M4', 'RS-M4-04', 'research', 'Validate 18 agent perspectives load',    NULL,       'queued'),
  ('M4', 'CD-M4-05', 'code',     'Router agent classifies test prompts',   'RS-M4-04', 'queued'),
  ('M4', 'CD-M4-06', 'code',     'Subagent isolation verified',            'CD-M4-03', 'queued'),
  ('M4', 'CW-M4-07', 'cowork',   'Run /staff-review from Cowork',         'CD-M4-01', 'queued'),
  ('M4', 'CD-M4-08', 'code',     'Agent budget allocation tests pass',     'CD-M4-03', 'queued'),
  ('M4', 'RS-M4-09', 'research', 'Document coordinator-subagent patterns', 'CD-M4-06', 'queued'),
  ('M4', 'CD-M4-10', 'code',     'M4 milestone tests pass',               'ALL',       'queued');
```

## DSPy Signatures

```python
class SpawnAgentTeam(dspy.Signature):
    """Spawn a multi-agent review team with scoped tools per agent."""
    team_name: str = dspy.InputField()
    agent_specs: list[dict] = dspy.InputField(desc="AgentDefinition per specialist")
    pr_diff: str = dspy.InputField()
    findings: list[dict] = dspy.OutputField(desc="Aggregated findings with severity")
    pass_rate: float = dspy.OutputField(desc="% of agents that completed without error")

class ClassifyPromptComplexity(dspy.Signature):
    """Router: classify prompt → HAIKU / SONNET / OPUS."""
    prompt: str = dspy.InputField()
    model_tier: str = dspy.OutputField(desc="haiku | sonnet | opus")
    reasoning: str = dspy.OutputField()
```

## Tasks

### CD-M4-01: Staff review team tests
**PASS**: `cd staff-review-team && npm test` exits 0

### CD-M4-02: Skeptical codegen tests
**PASS**: `cd skeptical-codegen-team && npm test` exits 0

### CD-M4-03: Agent factory compiles
**PASS**: `cd packages/agent-factory && npx tsc --noEmit` exits 0

### RS-M4-04: 18 perspectives load
**PASS**: `ls .claude/agents/*.md | wc -l | xargs test 18 -eq`

### CD-M4-05: Router classifies correctly
**PASS**: Router returns `haiku` for "what's 2+2", `sonnet` for "refactor this function", `opus` for "redesign the auth architecture"

### CD-M4-06: Subagent isolation
**PASS**: No `AgentDefinition` has `inheritParentContext: true`

### CW-M4-07: /staff-review from Cowork
**Type**: `cowork:` | **Owner**: claude-architect
**PASS**: Staff review command completes in Cowork desktop with 6-agent fan-out

### CD-M4-08: Budget allocation
**PASS**: `npx vitest run packages/agent-factory/tests/budget.test.ts` exits 0

### RS-M4-09: Document patterns
**PASS**: `docs/coordinator-subagent-patterns.md` exists with diagrams

### CD-M4-10: M4 tests
**PASS**: `npx vitest run tests/milestones/m4-agents.test.ts` exits 0

## Sprint Gate

All 10 green → merge → tag `v0.4.0`.
