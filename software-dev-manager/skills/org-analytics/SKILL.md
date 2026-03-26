---
name: org-analytics
description: Org-level Claude Code analytics — adoption dashboards, contribution metrics, GitHub PR attribution, leaderboard, and ROI measurement.
argument-hint: "<adoption|contributions|leaderboard|roi|export>"
---

# Org Analytics — Adoption, Contributions & ROI

## Dashboards

| Plan | URL | Includes |
|------|-----|----------|
| Teams/Enterprise | claude.ai/analytics/claude-code | Usage, contributions, GitHub, leaderboard, CSV export |
| API (Console) | platform.claude.com/claude-code | Usage, spend, team insights |

## Key Metrics

| Metric | Description |
|--------|-------------|
| PRs with CC | Merged PRs containing Claude Code-assisted lines |
| Lines of code with CC | Lines written with Claude Code in merged PRs |
| PRs with CC (%) | % of all merged PRs with Claude Code |
| Suggestion accept rate | % of Edit/Write/NotebookEdit suggestions accepted |
| Lines accepted | Total lines accepted in sessions |
| DAU/WAU/MAU | Daily/weekly/monthly active users |

## GitHub Contribution Metrics Setup

1. Install Claude GitHub app: github.com/apps/claude
2. Claude Owner → claude.ai/admin-settings/claude-code → enable analytics
3. Enable "GitHub analytics" toggle
4. Authenticate GitHub → select orgs
5. Data appears within 24 hours

## PR Attribution

PRs labeled `claude-code-assisted` automatically when:
- PR contains lines matching Claude Code session output
- Conservative matching (high confidence only)
- Sessions from 21 days before to 2 days after merge date
- Lines normalized (whitespace, quotes, case)
- Code rewritten >20% not attributed

### Excluded from attribution
- Lock files (package-lock.json, yarn.lock, Cargo.lock)
- Generated code (protobuf, build artifacts, minified)
- Build dirs (dist/, node_modules/, target/)
- Lines over 1000 chars
- Test fixtures and snapshots

## Adoption Tracking (Supabase)

```sql
-- DAU/WAU/MAU
SELECT
  DATE(recorded_at) AS day,
  COUNT(DISTINCT labels->>'user.account_uuid') AS dau,
  COUNT(DISTINCT labels->>'user.account_uuid') FILTER (
    WHERE recorded_at > now() - INTERVAL '7 days'
  ) AS wau,
  COUNT(DISTINCT labels->>'user.account_uuid') FILTER (
    WHERE recorded_at > now() - INTERVAL '30 days'
  ) AS mau
FROM otel_metrics
WHERE metric_name = 'claude_code.session.count'
GROUP BY DATE(recorded_at)
ORDER BY day DESC;
```

## ROI Measurement

Track alongside existing engineering KPIs:

| KPI | Before CC | After CC | Delta |
|-----|-----------|----------|-------|
| PRs per user per week | baseline | measure | % change |
| Time to first PR (onboarding) | baseline | measure | days saved |
| Lines of code per session | — | measure | productivity |
| Code review turnaround | baseline | measure | hours saved |
| DORA: deploy frequency | baseline | measure | % change |
| DORA: lead time | baseline | measure | hours saved |

## Leaderboard Query

```sql
SELECT
  labels->>'user.email' AS email,
  SUM(CASE WHEN metric_name = 'claude_code.pull_request.count' THEN value ELSE 0 END)::int AS prs,
  SUM(CASE WHEN metric_name = 'claude_code.lines_of_code.count' THEN value ELSE 0 END)::int AS lines,
  SUM(CASE WHEN metric_name = 'claude_code.commit.count' THEN value ELSE 0 END)::int AS commits,
  ROUND(SUM(CASE WHEN metric_name = 'claude_code.cost.usage' THEN value ELSE 0 END)::numeric, 2) AS cost_usd
FROM otel_metrics
WHERE metric_name IN (
  'claude_code.pull_request.count',
  'claude_code.lines_of_code.count',
  'claude_code.commit.count',
  'claude_code.cost.usage'
)
GROUP BY labels->>'user.email'
ORDER BY prs DESC
LIMIT 20;
```

## Power User Identification

Find team members who can train others:
- Top 10 by PR count with Claude Code
- Highest suggestion accept rate
- Most active time (user + cli)
- Longest session streaks

## CSV Export

Teams/Enterprise: click "Export all users" on leaderboard for full contribution data.
