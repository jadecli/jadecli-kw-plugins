---
name: claude-code-security-review
description: Automated multi-agent PR security review. Fleet of specialized agents examine code for logic errors, security vulnerabilities, broken edge cases, and regressions.
argument-hint: "<setup|customize|pricing|trigger>"
---

# Claude Code Security Review

Multi-agent code review that posts inline findings on PRs. Specialized agents analyze diffs in context of your full codebase.

## How It Works

1. PR opens (or push, or `@claude review`)
2. Fleet of specialized agents analyze diff + surrounding code in parallel
3. Verification step filters false positives
4. Deduplicated findings posted as inline comments on specific lines
5. ~20 min average, scales with PR size

## Severity Levels

| Marker | Severity | Meaning |
|--------|----------|---------|
| 🔴 | Normal | Bug — fix before merging |
| 🟡 | Nit | Minor — worth fixing, not blocking |
| 🟣 | Pre-existing | Bug exists but not introduced by this PR |

Each finding includes collapsible extended reasoning.

## Setup

1. Go to [claude.ai/admin-settings/claude-code](https://claude.ai/admin-settings/claude-code)
2. Click **Setup** in Code Review section
3. Install Claude GitHub App (Contents, Issues, PRs: read & write)
4. Select repositories
5. Set review triggers per repo:
   - **Once after PR creation**: single review on open
   - **After every push**: review on each push, auto-resolves fixed issues
   - **Manual**: only on `@claude review` comment

## Trigger Manually

Comment on any PR (top-level, not inline):
```
@claude review
```

Must have owner/member/collaborator access. PR must be open, not draft.

## Customize Reviews

### CLAUDE.md

Standard project instructions. Violations flagged as nit-level findings. Works bidirectionally — if PR makes CLAUDE.md outdated, Claude flags that too.

### REVIEW.md

Review-only guidance at repo root:

```markdown
# Code Review Guidelines

## Always check
- New API endpoints have integration tests
- Database migrations are backward-compatible
- Error messages don't leak internal details

## Style
- Prefer match statements over chained isinstance
- Use structured logging, not f-string interpolation

## Skip
- Generated files under src/gen/
- Formatting-only changes in *.lock files
```

## Pricing

- $15-25 average per review
- Scales with PR size and codebase complexity
- Billed through extra usage (not plan included)
- "After every push" multiplies cost per push count
- Set spend cap at [claude.ai/admin-settings/usage](https://claude.ai/admin-settings/usage)

## Analytics

View at [claude.ai/analytics/code-review](https://claude.ai/analytics/code-review):
- PRs reviewed (daily)
- Cost (weekly)
- Feedback (auto-resolved comments)
- Repository breakdown

## Requirements

- Teams or Enterprise subscription
- Not available with Zero Data Retention
- Admin enables at claude.ai/admin-settings/claude-code

## vs GitHub Actions

| | Code Review (managed) | GitHub Actions (self-hosted) |
|---|---|---|
| Runs on | Anthropic infrastructure | GitHub runners |
| Setup | Admin toggle | Workflow YAML |
| Multi-agent | Yes (fleet) | Single agent |
| Cost | $15-25/review | API tokens + runner minutes |
| Customization | CLAUDE.md + REVIEW.md | Full workflow control |
