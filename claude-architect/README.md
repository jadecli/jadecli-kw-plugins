# Distinguished Claude Architect

Org-level architecture strategy for jadecli. Decomposes the Claude Certified Architect Foundations exam and the Enterprise LLM Architecture Playbook into **directives** (rules), **primitives** (building blocks), **skills** (domain capabilities), and **milestones** (build-passing checkpoints).

If all milestones build, the entire jadecli ecosystem is validated end-to-end:
- All 8 knowledge worker plugins operational with MCPs
- Supabase as the data layer (OTel events, Cube.js semantic models)
- Netlify running jadecli.com
- Linear for task management and product roadmap
- iOS mobile working with Slack and Cowork integrations
- Dispatch, Remote Control, Channels all functional
- GitHub Actions, code review, security review green
- Real data flowing — not mocks

## Decomposition

### Directives (architectural rules)

Non-negotiable principles that govern every decision. Violations fail the build.

### Primitives (building blocks)

The fundamental tools, protocols, and patterns available to all knowledge workers.

### Skills (5 exam domains)

| Domain | Weight | Skill |
|--------|--------|-------|
| D1: Agentic Architecture & Orchestration | 27% | `d1-agentic-orchestration` |
| D2: Tool Design & MCP Integration | 18% | `d2-tool-design-mcp` |
| D3: Claude Code Configuration & Workflows | 20% | `d3-claude-code-workflows` |
| D4: Prompt Engineering & Structured Output | 20% | `d4-prompt-structured-output` |
| D5: Context Management & Reliability | 15% | `d5-context-reliability` |

### Milestones (10 build checkpoints)

| # | Milestone | Validates |
|---|-----------|-----------|
| M1 | Foundation | OTel → Supabase pipeline, CLAUDE.md hierarchy, auth policy |
| M2 | Tools & MCP | All MCP servers connected, tool descriptions correct, error handling |
| M3 | Plugins | All 8 KW plugins installed, skills invocable, marketplace synced |
| M4 | Agents | Staff review + skeptical codegen teams pass, agent factory spawns |
| M5 | Integrations | Slack @Claude, Channels (Telegram/Discord/iMessage), Remote Control, Dispatch |
| M6 | CI/CD | GitHub Actions green, code review posting, security review passing |
| M7 | Data | Supabase tables populated with real OTel events, Cube.js models queryable |
| M8 | Frontend | Netlify deploying jadecli.com, telemetry dashboard rendering real data |
| M9 | Mobile | iOS Cowork + Slack + Dispatch end-to-end, permission relay working |
| M10 | Roadmap | Linear integration, product roadmap tasks tracked, WBR generating |

## Certification Reference

- Exam: https://anthropic.skilljar.com/claude-certified-architect-foundations-access-request
- Passing score: 720/1000
- `architecture-cert.pdf` — exam guide with domains, task statements, scenarios
- `architecture-playbook.pdf` — Enterprise LLM Architecture patterns and anti-patterns
