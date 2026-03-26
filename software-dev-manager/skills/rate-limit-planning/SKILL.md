---
name: rate-limit-planning
description: TPM/RPM capacity planning by org size. Concurrent usage modeling, rate limit recommendations, and scaling guidance for Claude Code teams.
argument-hint: "<plan|current|scale> [team-size]"
---

# Rate Limit Planning — Capacity by Org Size

## TPM/RPM Recommendations

| Team Size | TPM per User | RPM per User | Total TPM (example) |
|-----------|-------------|-------------|-------------------|
| 1-5 | 200k-300k | 5-7 | 1.5M |
| 5-20 | 100k-150k | 2.5-3.5 | 2M |
| 20-50 | 50k-75k | 1.25-1.75 | 2.5M |
| 50-100 | 25k-35k | 0.62-0.87 | 3M |
| 100-500 | 15k-20k | 0.37-0.47 | 4M |
| 500+ | 10k-15k | 0.25-0.35 | 7.5M |

TPM per user decreases with team size — fewer users concurrent in larger orgs. Limits are org-level, not per-user, so individuals can burst.

## Why It Decreases

- Not all users are active simultaneously
- Peak concurrency ~20-30% of total users
- Rate limits are shared pools, not per-seat allocations
- Individual burst above calculated share is fine when others are idle

## Capacity Planning Query

```sql
-- Peak concurrent users (5-min windows)
SELECT
  DATE_TRUNC('hour', recorded_at) +
    (EXTRACT(MINUTE FROM recorded_at)::int / 5 * INTERVAL '5 minutes') AS window,
  COUNT(DISTINCT labels->>'user.account_uuid') AS concurrent_users,
  SUM(value)::bigint AS tokens_in_window
FROM otel_metrics
WHERE metric_name = 'claude_code.token.usage'
  AND recorded_at > now() - INTERVAL '7 days'
GROUP BY window
ORDER BY concurrent_users DESC
LIMIT 20;
```

## Actual vs Provisioned

```sql
-- Daily peak TPM
SELECT
  DATE(recorded_at) AS day,
  MAX(tokens_per_minute) AS peak_tpm
FROM (
  SELECT
    DATE_TRUNC('minute', recorded_at) AS minute,
    DATE(recorded_at) AS recorded_at,
    SUM(value)::bigint AS tokens_per_minute
  FROM otel_metrics
  WHERE metric_name = 'claude_code.token.usage'
  GROUP BY DATE_TRUNC('minute', recorded_at), DATE(recorded_at)
) sub
GROUP BY day
ORDER BY day DESC
LIMIT 30;
```

## Scale Triggers

| Signal | Action |
|--------|--------|
| 429 errors increasing | Raise TPM limit |
| Peak TPM > 80% of limit | Request increase proactively |
| New team onboarding | Pre-allocate per table above |
| Agent teams enabled | Multiply per-user TPM by 3-5x for team users |
| Training sessions | Temporarily increase (all users concurrent) |

## Agent Teams Impact

Agent teams spawn multiple Claude instances. Plan for:
- Standard user: 1x TPM allocation
- Agent team user: 5-7x TPM allocation
- Staff review team (6 agents): ~7x per invocation

## Special Scenarios

| Scenario | Recommendation |
|----------|---------------|
| Live training (all users active) | 3x normal TPM for duration |
| Hackathon | 5x normal TPM, temporary |
| CI/CD automation | Separate workspace with own limits |
| Scheduled tasks (cron) | Off-peak, stagger across minutes |

## Request Limit Increase

1. Calculate: `users × TPM_per_user = total_TPM`
2. Add 30% headroom for burst
3. Request via Anthropic Console or account team
4. Monitor actual usage after rollout, adjust quarterly
