# Software Dev Manager Plugin

Org-level Claude Code usage monitoring, cost management, and knowledge worker analytics for jadecli. Replicates the Amazon Ads WAF well-architected pattern — but for Claude token spend, team adoption, and knowledge worker ROI.

## Framework

Six components mirroring WAF, adapted for Claude Code orgs:

1. **OTel Setup** — foundational telemetry pipeline (Supabase + OpenTelemetry)
2. **Usage Monitoring** — token, session, tool, and code metrics per user/team
3. **Cost Management** — budget controls, spend limits, model selection optimization
4. **Org Analytics** — adoption dashboards, contribution metrics, GitHub integration
5. **Knowledge Worker Metrics** — per-role productivity (data eng, frontend, integration eng)
6. **Rate Limit Planning** — TPM/RPM capacity planning by team size

## Skills

| Skill | Description |
|-------|-------------|
| `otel-setup` | Configure OpenTelemetry export to Supabase — metrics, events, managed settings |
| `usage-monitoring` | Track tokens, sessions, tools, lines of code, commits, PRs per user/team |
| `cost-management` | Budget controls, spend limits, model selection, thinking effort optimization |
| `org-analytics` | Adoption dashboards, contribution metrics, leaderboard, GitHub PR attribution |
| `knowledge-worker-metrics` | Per-role analytics — measure ROI across knowledge worker plugins |
| `rate-limit-planning` | TPM/RPM capacity planning by org size, concurrent usage modeling |

## Data Flow

```
Claude Code (developer machines)
  → OpenTelemetry export (OTLP/gRPC)
    → OTel Collector
      → Supabase PostgreSQL (otel_events, otel_metrics)
        → Dashboards (Next.js / Cube.js semantic layer)
```

## Requires

- `CLAUDE_CODE_ENABLE_TELEMETRY=1` on all developer machines
- `SUPABASE_URL` and `SUPABASE_SERVICE_ROLE_KEY`
- OTel Collector forwarding to Supabase (or direct OTLP export)
