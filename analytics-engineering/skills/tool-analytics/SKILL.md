---
name: tool-analytics
description: Analyze Claude Code tool call patterns — success rates, duration, error frequency, and tool usage rankings from Supabase otel_events.
argument-hint: "<breakdown|errors|slowest|by-session>"
---

# Tool Analytics — Performance & Error Patterns

Analyze `claude_code.tool_result` events from Supabase.

## Queries

### Tool Breakdown (success rate ranked)

```sql
SELECT
  attributes->>'tool_name' AS tool_name,
  COUNT(*)::int AS count,
  ROUND(
    AVG(CASE WHEN (attributes->>'success')::boolean THEN 1 ELSE 0 END)::numeric, 4
  )::float AS success_rate
FROM otel_events
WHERE event_name = 'claude_code.tool_result'
GROUP BY attributes->>'tool_name'
ORDER BY count DESC;
```

### Error-Prone Tools

```sql
SELECT
  attributes->>'tool_name' AS tool,
  COUNT(*) FILTER (WHERE NOT (attributes->>'success')::boolean) AS errors,
  COUNT(*) AS total,
  ROUND(
    COUNT(*) FILTER (WHERE NOT (attributes->>'success')::boolean)::numeric / COUNT(*)::numeric, 4
  ) AS error_rate
FROM otel_events
WHERE event_name = 'claude_code.tool_result'
GROUP BY attributes->>'tool_name'
HAVING COUNT(*) FILTER (WHERE NOT (attributes->>'success')::boolean) > 0
ORDER BY error_rate DESC;
```

### Average Duration by Tool

```sql
SELECT
  attributes->>'tool_name' AS tool,
  COUNT(*)::int AS calls,
  ROUND(AVG((attributes->>'duration_ms')::float)::numeric, 1) AS avg_ms,
  ROUND(MAX((attributes->>'duration_ms')::float)::numeric, 1) AS max_ms,
  ROUND(PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY (attributes->>'duration_ms')::float)::numeric, 1) AS p95_ms
FROM otel_events
WHERE event_name = 'claude_code.tool_result'
  AND attributes->>'duration_ms' IS NOT NULL
GROUP BY attributes->>'tool_name'
ORDER BY avg_ms DESC;
```

### Tool Usage Over Time (daily)

```sql
SELECT
  DATE(recorded_at) AS day,
  attributes->>'tool_name' AS tool,
  COUNT(*) AS calls
FROM otel_events
WHERE event_name = 'claude_code.tool_result'
GROUP BY DATE(recorded_at), attributes->>'tool_name'
ORDER BY day DESC, calls DESC;
```

### Tools by Session

```sql
SELECT
  session_id,
  attributes->>'tool_name' AS tool,
  COUNT(*) AS calls,
  COUNT(*) FILTER (WHERE NOT (attributes->>'success')::boolean) AS errors
FROM otel_events
WHERE event_name = 'claude_code.tool_result'
GROUP BY session_id, attributes->>'tool_name'
ORDER BY calls DESC
LIMIT 50;
```

## Dashboard Component

```tsx
function ToolRow({ tool }: { tool: ToolBreakdown }) {
  const pct = (tool.success_rate * 100).toFixed(1);
  return (
    <tr className="border-b border-white/5">
      <td className="px-3 py-2 text-sm text-white">{tool.tool_name}</td>
      <td className="px-3 py-2 text-center text-xs text-neutral-300">{tool.count}</td>
      <td className="py-2 text-right text-xs">
        <span className={
          tool.success_rate >= 0.95 ? "text-green-400" :
          tool.success_rate >= 0.8 ? "text-yellow-400" : "text-red-400"
        }>
          {pct}%
        </span>
      </td>
    </tr>
  );
}
```
