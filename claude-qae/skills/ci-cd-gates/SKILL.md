---
name: ci-cd-gates
description: CI quality gates — what blocks merge, what warns. GitHub Actions workflow checks and their pass/fail criteria.
argument-hint: "<blocking|warning|check>"
---

# CI/CD Quality Gates

## Blocking (merge blocked if any fail)

| Gate | Workflow | What It Checks |
|------|----------|----------------|
| TypeScript build | ci.yml | `tsc` compiles all packages |
| Vitest suite | ci.yml | All workspace tests pass |
| Python tests | ci.yml | `pytest` passes |
| Bats tests | ci.yml | 48 org-ops tests pass |
| Commitlint | husky pre-commit | PR title follows conventional commits |
| Secrets scan | ci.yml | No API keys or tokens in code |
| Shellcheck | ci.yml | All `.sh` scripts lint clean |
| Branch protection | repo settings | `ci` check required, enforce_admins |

## Warning (review recommended, doesn't block)

| Gate | Workflow | What It Checks |
|------|----------|----------------|
| Code review | claude-code-review.yml | Multi-agent review findings posted |
| Bloom eval | bloom-eval.yml | Prompt quality regression detected |
| Principle checks | principle-checks.yml | Architecture principle violations |

## Auth Gate (blocks if violated)

```yaml
# Every workflow must use:
claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}

# Never:
anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

## PR Checklist (from template)

- [ ] Conventional commit title
- [ ] < 200 lines added
- [ ] Tests for new functionality
- [ ] No `ANTHROPIC_API_KEY`
- [ ] Branch rebased onto main
- [ ] CI green

## Neon Branch-per-PR

Every PR gets a Neon database branch for preview testing. Workflow: `neon-branch.yml`. Created on PR open, deleted on PR close.

## Deploy Gates

| Environment | Gate | Trigger |
|-------------|------|---------|
| Vercel preview | Auto on PR | apps/console changes |
| Vercel production | CI green on main | Push to main |
| Netlify jadecli.com | CI green on main | Push to main |
| Stainless SDK gen | Release-please tag | Version bump |
