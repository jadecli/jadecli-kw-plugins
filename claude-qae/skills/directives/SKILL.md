---
name: directives
description: QAE quality rules — what must never ship. Test coverage requirements, bug severity definitions, release gates.
argument-hint: "<list|check>"
---

# QAE Directives

## Test Coverage

- **Q-COV-1**: No PR merges without tests for new functionality or regression tests for bug fixes.
- **Q-COV-2**: Minimum 80% line coverage for packages with `vitest.config.ts`.
- **Q-COV-3**: Every MCP tool has at least 1 happy-path and 1 error-path test.
- **Q-COV-4**: Every milestone condition from the architect has a corresponding automated test.

## Bug Severity

- **Q-SEV-1**: P0 (data loss, security, auth bypass) — blocks release, fix within 4 hours.
- **Q-SEV-2**: P1 (feature broken, integration down) — blocks release, fix within 24 hours.
- **Q-SEV-3**: P2 (degraded experience, workaround exists) — fix within sprint.
- **Q-SEV-4**: P3 (cosmetic, minor) — backlog, fix when convenient.

## Release Gates

- **Q-GATE-1**: All CI checks green. No exceptions.
- **Q-GATE-2**: No P0 or P1 bugs open.
- **Q-GATE-3**: `npm test` passes in all workspace packages.
- **Q-GATE-4**: Conventional commit title on every PR.
- **Q-GATE-5**: Squash merges only — no merge commits.
- **Q-GATE-6**: `ANTHROPIC_API_KEY` never appears in code, env, or secrets.

## Data Quality

- **Q-DATA-1**: No mock data in staging or production. Real OTel events only.
- **Q-DATA-2**: Supabase RLS enabled on every table — tested in migration.
- **Q-DATA-3**: Kimball DDL files have pipeline metadata headers — validated in CI.
- **Q-DATA-4**: Cube.js models validate against actual Supabase schema.
