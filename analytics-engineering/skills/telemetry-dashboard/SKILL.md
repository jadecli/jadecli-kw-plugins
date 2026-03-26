---
name: telemetry-dashboard
description: Build and query the telemetry dashboard for Claude Code agent sessions. Session stats, tool breakdown, cost cards, and error rates from Supabase otel_events.
argument-hint: "<build|query|refresh>"
---

# Telemetry Dashboard

React dashboard components backed by Supabase OTel event data.

## Components

### StatCard

```tsx
function StatCard({ label, value }: { label: string; value: string }) {
  return (
    <div className="rounded-lg border border-white/10 bg-white/5 px-5 py-4">
      <p className="text-xs uppercase tracking-wide text-neutral-500">{label}</p>
      <p className="mt-1 text-2xl font-semibold text-white">{value}</p>
    </div>
  );
}
```

### Dashboard Layout

```tsx
export function TelemetryDashboard({
  summary,
  sessions,
  toolBreakdown,
}: {
  summary: TelemetrySummary;
  sessions: SessionEvent[];
  toolBreakdown: ToolBreakdown[];
}) {
  // Stat cards row: total sessions, total tool calls, error rate
  // Sessions table: session_id, agent_type, recorded_at, tool_count
  // Tool breakdown: tool_name, count, success_rate (color-coded)
}
```

### Server Page

```tsx
export const dynamic = "force-dynamic";

export default async function TelemetryPage() {
  const [summary, sessions, toolBreakdown] = await Promise.all([
    fetchSummary(),
    fetchSessions(),
    fetchToolBreakdown(),
  ]);
  return <TelemetryDashboard summary={summary} sessions={sessions} toolBreakdown={toolBreakdown} />;
}
```

## Types

```typescript
interface SessionEvent {
  session_id: string;
  agent_type: string;
  recorded_at: string;
  tool_count: number;
}

interface TelemetrySummary {
  totalSessions: number;
  totalToolCalls: number;
  errorRate: number;
}

interface ToolBreakdown {
  tool_name: string;
  count: number;
  success_rate: number;
}

interface CostSummary {
  totalCostUsd: number;
  totalTokens: number;
  totalInputTokens: number;
  totalOutputTokens: number;
  sessionCount: number;
  avgCostPerSession: number;
  costByModel: { model: string; cost: number; tokens: number; sessions: number }[];
}
```

## Supabase Client

```typescript
import { createClient } from "@supabase/supabase-js";

const supabase = createClient(
  process.env.SUPABASE_URL!,
  process.env.SUPABASE_SERVICE_ROLE_KEY!
);
```
