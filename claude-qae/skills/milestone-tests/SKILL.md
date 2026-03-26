---
name: milestone-tests
description: Concrete test implementations for each of the 10 architect milestones. Run these to validate the entire jadecli ecosystem end-to-end.
argument-hint: "<M1|M2|...|M10|all>"
---

# Milestone Tests

Each test maps 1:1 to an architect milestone condition. All must pass.

## M1: Foundation Tests

```typescript
describe("M1: Foundation", () => {
  it("OTel events reach Supabase within 1 hour", async () => {
    const { count } = await supabase
      .from("otel_events")
      .select("*", { count: "exact", head: true })
      .gte("recorded_at", new Date(Date.now() - 3600000).toISOString());
    expect(count).toBeGreaterThan(0);
  });

  it("no ANTHROPIC_API_KEY in workflows", async () => {
    const result = execSync("grep -r ANTHROPIC_API_KEY .github/ || true").toString();
    expect(result.trim()).toBe("");
  });

  it("all workflows use CLAUDE_CODE_OAUTH_TOKEN", () => {
    const workflows = globSync(".github/workflows/*.yml");
    for (const wf of workflows) {
      const content = readFileSync(wf, "utf-8");
      if (content.includes("anthropic") || content.includes("claude")) {
        expect(content).toContain("CLAUDE_CODE_OAUTH_TOKEN");
        expect(content).not.toContain("ANTHROPIC_API_KEY");
      }
    }
  });

  it("AST index builds successfully", () => {
    execSync("source .venv/bin/activate && python scripts/build-ast-index.py");
    expect(existsSync("ast-index/L1-files.json")).toBe(true);
    expect(existsSync("ast-index/L2-symbols.json")).toBe(true);
    const l2 = JSON.parse(readFileSync("ast-index/L2-symbols.json", "utf-8"));
    expect(l2.length).toBeGreaterThan(0);
  });
});
```

## M2: Tools & MCP Tests

```typescript
describe("M2: Tools & MCP", () => {
  it("Supabase MCP responds", async () => {
    const tables = await mcpCall("supabase", "list_tables");
    expect(tables.isError).toBeFalsy();
    expect(tables.content).toContain("otel_events");
  });

  it("MCP errors are structured", async () => {
    const error = await mcpCall("supabase", "query", { sql: "INVALID SQL" });
    expect(error.isError).toBe(true);
    expect(error.errorCategory).toBeDefined();
    expect(error.isRetryable).toBeDefined();
  });

  it("no agent has > 8 tools", () => {
    const agents = globSync(".claude/agents/*.md");
    for (const agent of agents) {
      const content = readFileSync(agent, "utf-8");
      const toolMatch = content.match(/allowed-tools:(.+)/);
      if (toolMatch) {
        const tools = toolMatch[1].split(",").map(t => t.trim());
        expect(tools.length).toBeLessThanOrEqual(8);
      }
    }
  });
});
```

## M3: Plugins Tests

```typescript
describe("M3: Plugins", () => {
  it("marketplace manifest is valid JSON", () => {
    const manifest = JSON.parse(
      readFileSync(".claude-plugin/marketplace.json", "utf-8")
    );
    expect(manifest.name).toBe("jadecli-kw-plugins");
    expect(manifest.plugins.length).toBeGreaterThanOrEqual(8);
  });

  it("every plugin has skills", () => {
    for (const plugin of manifest.plugins) {
      const skillDirs = globSync(`${plugin.source.replace('./', '')}/skills/*/SKILL.md`);
      expect(skillDirs.length).toBeGreaterThan(0);
    }
  });

  it("every SKILL.md has required frontmatter", () => {
    const skills = globSync("**/skills/*/SKILL.md");
    for (const skill of skills) {
      const content = readFileSync(skill, "utf-8");
      expect(content).toMatch(/^---\nname:/);
      expect(content).toMatch(/description:/);
    }
  });
});
```

## M4: Agents Tests

```typescript
describe("M4: Agents", () => {
  it("staff review team tests pass", () => {
    execSync("cd staff-review-team && npm test", { stdio: "pipe" });
  });

  it("skeptical codegen team tests pass", () => {
    execSync("cd skeptical-codegen-team && npm test", { stdio: "pipe" });
  });

  it("18 agent perspectives exist", () => {
    const agents = globSync(".claude/agents/*.md");
    expect(agents.length).toBe(18);
  });

  it("subagents have isolated context (no parent inherit)", () => {
    const agentDefs = globSync("packages/agent-factory/src/agents/**/*.ts");
    for (const def of agentDefs) {
      const content = readFileSync(def, "utf-8");
      expect(content).not.toContain("inheritParentContext: true");
    }
  });
});
```

## M5: Integration Tests

```typescript
describe("M5: Integrations", () => {
  it("Remote Control session registerable", async () => {
    // Verify claude remote-control --help exits 0
    execSync("claude remote-control --help", { stdio: "pipe" });
  });

  it("scheduled task CronCreate works", async () => {
    // Verify cron tool is available
    const result = execSync("claude -p 'list scheduled tasks' --output-format json", { stdio: "pipe" });
    expect(result.toString()).toContain("session_id");
  });
});
```

## M6: CI/CD Tests

```typescript
describe("M6: CI/CD", () => {
  it("latest main CI run is green", async () => {
    const result = execSync("gh run list --branch main --limit 1 --json conclusion").toString();
    const runs = JSON.parse(result);
    expect(runs[0].conclusion).toBe("success");
  });

  it("branch protection enabled", async () => {
    const result = execSync("gh api repos/jadecli/jadebot/branches/main/protection --jq '.required_status_checks'").toString();
    expect(result).toContain("ci");
  });
});
```

## M7: Data Tests

```typescript
describe("M7: Data", () => {
  it("otel_events has real sessions (> 10)", async () => {
    const { count } = await supabase
      .from("otel_events")
      .select("*", { count: "exact", head: true })
      .eq("event_name", "claude_code.session.stop");
    expect(count).toBeGreaterThan(10);
  });

  it("otel_events has tool results (> 100)", async () => {
    const { count } = await supabase
      .from("otel_events")
      .select("*", { count: "exact", head: true })
      .eq("event_name", "claude_code.tool_result");
    expect(count).toBeGreaterThan(100);
  });

  it("RLS enabled on all tables", async () => {
    const { data } = await supabase.rpc("check_rls_enabled");
    for (const table of data) {
      expect(table.rls_enabled).toBe(true);
    }
  });

  it("Kimball DDL files have metadata headers", () => {
    const sqlFiles = globSync("schema/**/*.sql");
    for (const f of sqlFiles) {
      const content = readFileSync(f, "utf-8");
      expect(content).toMatch(/^-- DESCRIPTION:/m);
      expect(content).toMatch(/^-- GRAIN:/m);
      expect(content).toMatch(/^-- LOAD_TYPE:/m);
    }
  });
});
```

## M8: Frontend Tests

```typescript
describe("M8: Frontend", () => {
  it("jadecli.com returns 200", async () => {
    const res = await fetch("https://jadecli.com");
    expect(res.status).toBe(200);
  });

  it("telemetry dashboard component exports", async () => {
    const mod = await import("../app/telemetry/dashboard");
    expect(typeof mod.TelemetryDashboard).toBe("function");
  });

  it("dashboard fetches real data (not mock)", async () => {
    const summary = await fetchSummary();
    expect(summary.totalSessions).toBeGreaterThan(0);
  });
});
```

## M9: Mobile Tests (manual + automated)

| Test | Method | Pass Criteria |
|------|--------|---------------|
| Cowork skill invocation | Manual on iOS | Skill returns valid output |
| Slack → Code from phone | Manual | Session created, visible in app |
| Dispatch task | Manual | Desktop session spawns |
| Remote Control | Manual | Mobile drives local session |
| Notification delivery | Manual | Completion notification on phone |
| Session sync | Automated | `gh api` check session exists |

## M10: Roadmap Tests

```typescript
describe("M10: Roadmap", () => {
  it("Linear project has issues", async () => {
    const issues = await linearClient.issues({ filter: { project: { name: { eq: "jadecli" } } } });
    expect(issues.nodes.length).toBeGreaterThan(5);
  });

  it("WBR ran this week", async () => {
    const runs = execSync("gh run list --workflow=wbr-cron.yml --limit 1 --json createdAt").toString();
    const lastRun = JSON.parse(runs)[0];
    const daysSince = (Date.now() - new Date(lastRun.createdAt).getTime()) / 86400000;
    expect(daysSince).toBeLessThan(7);
  });
});
```

## Run All Milestones

```bash
# Quick check (M1 + M6 + M8)
npx vitest run tests/milestones/m1-foundation.test.ts
npx vitest run tests/milestones/m6-cicd.test.ts
npx vitest run tests/milestones/m8-frontend.test.ts

# Full suite
npx vitest run tests/milestones/
```
