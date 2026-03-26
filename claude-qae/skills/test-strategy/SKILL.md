---
name: test-strategy
description: Test pyramid, coverage targets, and testing patterns for each domain in the jadecli ecosystem.
argument-hint: "<domain|coverage|pyramid|pattern>"
---

# Test Strategy

## Test Pyramid

```
         E2E (5%)
        /         \
    Integration (25%)
   /                  \
     Unit (70%)
```

## Coverage Targets by Package

| Package | Target | Framework | Key Tests |
|---------|--------|-----------|-----------|
| jade-enterprise-sdk | 85% | vitest | 378 tests — types, contracts, endpoints |
| jade-orchestration | 80% | vitest | 329 tests — team patterns, S-Team, cowork plugins |
| agent-factory | 75% | vitest | Agent spawning, budget allocation |
| staff-review-team | 80% | vitest | 6-agent fan-out, aggregation, severity classification |
| skeptical-codegen-team | 80% | vitest | Adversarial detection, dead code, logic errors |
| apps/console | 70% | vitest 4.x | Dashboard rendering, API routes |
| crawler | 75% | pytest | Spider output, pipeline transforms, cache |
| org-ops scripts | 90% | bats | 48 tests — idempotent mutations, OK/SKIP/FAIL format |

## Testing Patterns by Domain

### Agent Tests
- Mock Claude API responses (don't call real API in unit tests)
- Verify tool selection given specific context
- Test coordinator delegation logic with stubbed subagent results
- Validate handoff payload structure

### MCP Tests
- Happy path: tool call returns expected data
- Error path: tool call returns `{isError: true, errorCategory, isRetryable}`
- Timeout: tool call exceeds timeout → structured error
- Auth failure: expired token → 401 → structured error

### Plugin Tests
- Marketplace manifest validates against JSON schema
- Each skill SKILL.md has required frontmatter (name, description)
- Skills invoke without crash (smoke test)
- Plugin install/uninstall lifecycle

### Data Tests
- Supabase schema matches Cube.js model definitions
- RLS policies exist on all tables
- DDL files have pipeline metadata headers
- Kimball grain declarations present in all fact tables

### Integration Tests
- OTel events flow from Claude Code → Collector → Supabase
- GitHub Actions trigger on correct events
- Slack @Claude creates web session (E2E)
- Scheduled task fires on cron schedule

## TDD Workflow

```
1. Write failing test (RED)
2. Implement minimum to pass (GREEN)
3. Refactor while tests stay green (REFACTOR)
4. Commit: `test(scope): add tests for X` then `feat(scope): implement X`
```
