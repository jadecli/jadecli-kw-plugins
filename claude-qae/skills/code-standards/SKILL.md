---
name: code-standards
description: TypeScript, Python, SQL, and Bash code standards enforced via lint, hooks, and CI. Based on cert exam Domain 3 + playbook patterns.
argument-hint: "<typescript|python|sql|bash|check>"
---

# Code Standards

## TypeScript
- Strict mode enabled, ES modules
- Zod for runtime validation (3.x most packages, 4.x console)
- Conventional commits: `feat(scope):`, `fix(scope):`, `chore(scope):`
- XML-structured I/O for agent communication
- No `any` types — use `unknown` + type guards
- Prefer `const` assertions and discriminated unions

## Python
- Python 3.12+, type hints required (Pyright strict)
- Ruff for linting + formatting
- `set -euo pipefail` in all bash scripts called from Python
- `PYTHONDONTWRITEBYTECODE=1`, `PYTHONUNBUFFERED=1`

## SQL (Kimball)
- 1 table per file in `schema/facts/` and `schema/dimensions/`
- Pipeline metadata header required on every file:
  ```sql
  -- DESCRIPTION: what this table represents
  -- GRAIN: one row per ...
  -- LOAD_TYPE: append-only | SCD1 | SCD2 | full-refresh | incremental
  -- SOURCE: source table or system
  -- SCHEDULE: cron or frequency
  -- DEPENDS_ON: upstream tables
  ```
- Surrogate keys (SERIAL/BIGSERIAL), never business keys as PKs
- RLS enabled on every Supabase table
- Idempotent loads using event_id for dedup

## Bash
- `set -euo pipefail` on every script
- shellcheck clean (no warnings)
- OK/SKIP/FAIL output format for org-ops scripts
- Mock `gh` in tests — no real API calls

## Hooks Enforcement
- Pre-commit: commitlint (conventional title)
- Pre-push: `npm test` must pass
- PreToolUse: plan-gate.sh gates file creation
- PostToolUse: test-runner.sh auto-runs tests after edits
