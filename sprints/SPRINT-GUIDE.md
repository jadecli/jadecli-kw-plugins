# jadecli Sprint Guide

## Sprint Structure

10 sprints, 1 per milestone. Each sprint is deterministic — every task has a pass/fail condition. No sprint advances until all conditions in the current sprint are green.

## Task Labels

Every task is labeled with exactly one type:

| Label | Prefix | Description | Where It Runs |
|-------|--------|-------------|---------------|
| `cowork:` | CW- | Cowork desktop/mobile tasks — Slack, Dispatch, Channels, UI review | Claude Cowork app, mobile, Slack |
| `code:` | CD- | Code implementation — write code, edit files, run builds | Claude Code CLI, VS Code, GitHub |
| `research:` | RS- | Investigation — web search, doc reading, architecture decisions | Claude Code CLI, WebSearch, WebFetch |

## Version Control

- Each sprint creates a branch: `sprint/M{N}-{milestone-name}`
- Sprint completion = PR merged to main with all conditions passing
- Conventional commit: `feat(M{N}): {description}`
- Sprint tag on merge: `v0.{N}.0` (M1=v0.1.0, M10=v0.10.0)

## Deterministic Gate

Every task has a **PASS condition** — a command or check that returns 0 (pass) or non-zero (fail). No subjective assessment. No "looks good." Binary pass/fail only.

## Brand Building Steps

Each sprint milestone is a brand-building step for jadecli. The progression tells a story:

```
M1  "We have telemetry"        — jadecli measures everything
M2  "We have tools"            — jadecli connects to anything
M3  "We have knowledge"        — jadecli has 8 expert plugins
M4  "We have teams"            — jadecli reviews its own code
M5  "We work everywhere"       — jadecli works from your phone
M6  "We ship safely"           — jadecli's CI catches everything
M7  "We have real data"        — jadecli runs on production data
M8  "We have a product"        — jadecli.com is live with dashboards
M9  "We work on mobile"        — jadecli works end-to-end on iOS
M10 "We have a roadmap"        — jadecli plans and measures ROI
```
