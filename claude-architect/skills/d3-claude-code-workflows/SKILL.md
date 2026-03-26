---
name: d3-claude-code-workflows
description: "Cert Domain 3 (20%): CLAUDE.md hierarchy, commands/skills config, path-scoped rules, plan mode vs direct execution, iterative refinement, CI/CD integration."
argument-hint: "<task-statement-number>"
---

# D3: Claude Code Configuration & Workflows (20%)

## 3.1 CLAUDE.md Hierarchy
- User-level: `~/.claude/CLAUDE.md` (personal, not shared)
- Project-level: `.claude/CLAUDE.md` or root `CLAUDE.md` (version controlled)
- Subdirectory: scoped to files under that path
- `@import` syntax for modular standards files
- `.claude/rules/` for topic-specific rules (alternative to monolithic CLAUDE.md)
- `/memory` command to verify which files are loaded

## 3.2 Custom Commands & Skills
- Commands: `.claude/commands/` (project-scoped, shared) or `~/.claude/commands/` (personal)
- Skills: `SKILL.md` with frontmatter (`context: fork`, `allowed-tools`, `argument-hint`)
- `context: fork` isolates verbose output from main session
- `allowed-tools` restricts tool access during skill execution
- Skills = on-demand (invoked when relevant); CLAUDE.md = always loaded

## 3.3 Path-Scoped Rules
- `.claude/rules/` files with `paths:` frontmatter containing glob patterns
- Rules load only when editing matching files — reduces irrelevant context
- Glob patterns apply conventions by file type regardless of location (`**/*.test.tsx`)
- Prefer path-scoped rules over subdirectory CLAUDE.md for cross-directory conventions

## 3.4 Plan Mode vs Direct Execution
- Plan mode: complex tasks, architectural decisions, multi-file changes, 45+ files
- Direct execution: single-file bug fixes, well-scoped changes
- Plan mode = safe exploration before committing; prevents costly rework
- Combine: plan for investigation → direct for implementation
- Explore subagent for verbose discovery, returning summary to main context

## 3.5 Iterative Refinement
- Input/output examples > prose descriptions for transformation requirements
- TDD: write tests first → share failures → iterate to green
- Interview pattern: Claude asks questions to surface considerations before implementing
- Multiple interacting issues in one message; sequential iteration for independent issues

## 3.6 CI/CD Integration
- `-p` flag for non-interactive mode in pipelines
- `--bare` skips auto-discovery — recommended for all CI/scripted usage
- `--output-format json` + `--json-schema` for machine-parseable structured output
- Session context isolation: separate instance for review (don't self-review)
- Include prior review findings when re-running after new commits
- Document testing standards in CLAUDE.md for CI-invoked Claude Code

## jadecli CI Pipeline
- `claude-code-action@v1` with `claude_code_oauth_token`
- `claude-code-review.yml`: PR review with code-review plugin
- `claude.yml`: @claude mentions in issues/PRs
- `wbr-cron.yml`: weekly business review (Monday 9am UTC)
- `bloom-eval.yml`: prompt evaluation on PR changes
