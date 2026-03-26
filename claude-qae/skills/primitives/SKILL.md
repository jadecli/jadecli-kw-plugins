---
name: primitives
description: Testing tools, frameworks, and patterns available to QAE.
argument-hint: "<list|describe> [tool-name]"
---

# QAE Primitives

## Test Frameworks

| Tool | Language | Use |
|------|----------|-----|
| vitest | TypeScript | Unit + integration tests (3.x packages, 4.x console) |
| bats | Bash | Org-ops script tests (48 tests in dotfiles) |
| pytest | Python | Crawler tests, agent-factory-python |
| fast-check | TypeScript | Property-based testing |

## Linting & Standards

| Tool | Purpose |
|------|---------|
| TypeScript strict mode | Type safety |
| commitlint | Conventional commit enforcement |
| husky hooks | Pre-commit, pre-push gates |
| shellcheck | Bash script linting |
| ruff | Python linting + formatting |
| Pyright | Python type checking |

## CI Tools

| Tool | Purpose |
|------|---------|
| GitHub Actions | Automated test/lint/build pipeline |
| claude-code-action@v1 | AI-powered PR review |
| turbo | Monorepo build orchestration with caching |
| actions/cache | Build artifact caching (.next/cache) |

## Test Patterns

| Pattern | When | Example |
|---------|------|---------|
| Unit | Pure functions, utilities | Token counting, schema validation |
| Integration | Cross-package, DB | Supabase queries, MCP tool calls |
| E2E | Full workflow | Plugin install → skill invoke → output verify |
| Snapshot | UI components | Dashboard rendering |
| Property | Invariants | "all events have session_id" |
| Contract | API boundaries | SDK type exports match spec |

## Verification Commands

```bash
yarn test                                    # all workspace tests
cd packages/jade-orchestration && npx vitest run  # single package
npx vitest run tests/specific.test.ts        # single file
make test                                    # bats org-ops tests
make health                                  # SDK coverage + tool health
make check                                   # verify symlinks
```
