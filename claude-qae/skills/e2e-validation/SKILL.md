---
name: e2e-validation
description: End-to-end validation flows across plugins, integrations, and mobile. Proves the full jadecli ecosystem works with real data.
argument-hint: "<flow-name|all>"
---

# E2E Validation Flows

## Flow 1: Developer Session → Telemetry → Dashboard

```
Developer runs `cc` in Ghostty
  → Claude Code session starts
  → OTel exports metrics + events
  → Collector forwards to Supabase
  → otel_events table populated
  → Telemetry dashboard shows session
  → Cost card shows real USD value
```

**Assert**: `TelemetrySummary.totalSessions` increments after session.

## Flow 2: PR → CI → Code Review → Merge

```
Developer pushes branch
  → PR created
  → claude-code-review.yml triggers
  → Multi-agent review posts findings
  → Developer addresses findings
  → CI green
  → Squash merge to main
  → Netlify redeploys
```

**Assert**: PR has `Claude Code Review` check run with inline comments.

## Flow 3: Plugin Install → Skill Invoke → Output

```
claude plugin marketplace update jadecli-kw-plugins
  → claude plugin install data-engineering@jadecli-kw-plugins
  → /data-engineering:ast-index
  → ast-index/ populated with L1-L4 JSON
```

**Assert**: `ast-index/L2-symbols.json` has > 0 entries after skill invocation.

## Flow 4: Slack → Code Session → PR

```
@Claude in Slack: "fix the failing test in auth.ts"
  → Claude Code web session created
  → Claude reads repo, identifies issue
  → Creates fix, runs tests
  → Posts status to Slack thread
  → "Create PR" button → PR opened
```

**Assert**: Slack thread contains "View Session" and "Create PR" buttons.

## Flow 5: Mobile Dispatch → Desktop → Complete

```
iOS Claude app: send task to Desktop
  → Desktop session spawns
  → Task executes against local files
  → Completion notification on phone
  → View results in claude.ai/code
```

**Assert**: Session appears in mobile app session list with "complete" status.

## Flow 6: Channel → Local Session → Reply

```
Telegram message to bot: "check deploy status"
  → Message arrives in running Claude Code session
  → Claude checks deploy, formats response
  → Reply sent back to Telegram
```

**Assert**: Reply appears in Telegram chat within 60 seconds.

## Flow 7: Scheduled Task → Report → Supabase

```
Cron fires nightly at 2am
  → AST index rebuilds
  → Package audit runs
  → Results stored in Supabase
  → Dashboard reflects updated data
```

**Assert**: `ast-index/L1-files.json` modified_at within last 24 hours.

## Flow 8: Linear → Sprint → WBR

```
Linear sprint has issues with status
  → Monday 9am: WBR cron fires
  → Claude generates weekly business review
  → Report includes per-role metrics from OTel
  → Report posted/stored
```

**Assert**: WBR contains non-zero PRs and cost data for current week.

## Validation Script

```bash
#!/bin/bash
set -euo pipefail

echo "=== E2E Validation ==="

# M1: Foundation
echo -n "M1 OTel pipeline... "
count=$(psql $SUPABASE_DB_URL -t -c "SELECT COUNT(*) FROM otel_events WHERE recorded_at > now() - INTERVAL '1 hour'")
[[ $count -gt 0 ]] && echo "PASS" || echo "FAIL"

# M7: Real data
echo -n "M7 Real sessions... "
sessions=$(psql $SUPABASE_DB_URL -t -c "SELECT COUNT(*) FROM otel_events WHERE event_name = 'claude_code.session.stop'")
[[ $sessions -gt 10 ]] && echo "PASS ($sessions)" || echo "FAIL ($sessions)"

# M8: Frontend
echo -n "M8 jadecli.com... "
status=$(curl -s -o /dev/null -w '%{http_code}' https://jadecli.com)
[[ $status -eq 200 ]] && echo "PASS" || echo "FAIL ($status)"

echo "=== Done ==="
```
