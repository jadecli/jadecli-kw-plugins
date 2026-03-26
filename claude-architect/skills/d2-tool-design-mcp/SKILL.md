---
name: d2-tool-design-mcp
description: "Cert Domain 2 (18%): Tool descriptions, MCP structured errors, tool distribution, MCP server integration, built-in tool selection."
argument-hint: "<task-statement-number>"
---

# D2: Tool Design & MCP Integration (18%)

## 2.1 Tool Descriptions
- Descriptions are the primary mechanism for tool selection
- Include: input formats, example queries, edge cases, boundary explanations
- Ambiguous/overlapping descriptions cause misrouting (`analyze_content` vs `analyze_document`)
- Rename to differentiate: `analyze_content` → `extract_web_results`
- Split generic tools: `analyze_document` → `extract_data_points` + `summarize_content` + `verify_claim_against_source`

## 2.2 Structured MCP Errors
- `isError` flag communicates failure to agent
- Categories: transient (timeout), validation (invalid input), business (policy), permission
- Return `{isError, errorCategory, isRetryable, description}`
- Anti-pattern: generic "Operation failed" or throwing exceptions that crash the agent
- Distinguish access failures (retry) from valid empty results (no retry)

## 2.3 Tool Distribution
- 4-5 tools per subagent. 18 tools degrades reliability.
- Scoped access: only role-relevant tools
- Replace generic tools with constrained alternatives (`fetch_url` → `load_document` with URL validation)
- `tool_choice: "any"` guarantees a tool call (no conversational text)
- Forced selection: `{"type": "tool", "name": "extract_metadata"}` for required first steps

## 2.4 MCP Server Integration
- Project-level `.mcp.json` for shared team tools
- User-level `~/.claude.json` for personal/experimental
- Environment variable expansion: `${GITHUB_TOKEN}` — no committed secrets
- MCP resources expose content catalogs (schemas, issue summaries) to reduce exploratory calls
- Prefer community MCP servers for standard integrations (Jira, GitHub)

## 2.5 Built-in Tool Selection
- Grep: content search (function names, error messages, imports)
- Glob: file pattern matching (`**/*.test.tsx`)
- Read/Write: full file ops; Edit: targeted modifications with unique text matching
- Edit fails on non-unique match → fallback to Read + Write
- Build understanding incrementally: Grep → follow imports → Read specific functions

## jadecli MCP Inventory
- `@supabase/mcp-server-supabase` — data layer
- `@modelcontextprotocol/server-github` — PR, issues, repos
- `jade-console-mcp` — jadecli.com dashboard
- Cube.js API — semantic layer queries
- Linear MCP — task management
