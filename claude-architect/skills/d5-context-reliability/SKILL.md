---
name: d5-context-reliability
description: "Cert Domain 5 (15%): Context window management, long-session strategies, session resumption, error escalation, human-in-the-loop calibration."
argument-hint: "<task-statement-number>"
---

# D5: Context Management & Reliability (15%)

## 5.1 Context Window Management
- Tool results are the largest context consumers — prune before processing
- Application-side filter: extract only relevant fields, remove verbose data
- `/compact` with focus instructions to preserve what matters during summarization
- Move specialized instructions from CLAUDE.md to skills (load on-demand)
- Delegate verbose operations (tests, logs, docs) to subagents — summary returns to main

## 5.2 Long Session Strategies (Playbook)
- **Scratchpad pattern**: agent maintains structured file of key findings, architectural maps, decisions
- References scratchpad for subsequent questions instead of raw message history
- After 30+ min: accumulated token bloat causes inconsistent answers about early discoveries
- **Session compression**: summarize resolved turns into narrative, keep verbatim only for active issue

## 5.3 Session Resumption (Playbook)
- Filter out stale `tool_result` messages when resuming — force agent to re-fetch current data
- In dynamic environments: check which files changed since last session (git diff)
- `resume_session --update_context={files:['changed.ts'], changes:'description'}` for targeted re-analysis
- Starting fresh with injected summary > resuming with 4-hour-old tool results

## 5.4 Error Escalation & Human-in-the-Loop
- **Confidence calibration**: model outputs field-level confidence scores
- Automate extractions with confidence >90%; route <90% to human review queue
- Validate accuracy by document type and field — not just aggregate
- Escalation handoff: immediate for "I want a human NOW", context-gather first for complex issues
- Structured summary to human (not raw transcript): customer_id, root_cause, amount, recommended_action

## 5.5 Reliability Patterns
- Independent review instances > self-review (avoid reasoning context bias)
- Retry effective for formatting errors; ineffective for missing information — recognize when to fail fast
- Graceful tool failure: return error in tool result with `isError: true`, never throw exceptions
- Distinguish transient errors (retry) from valid empty results (no data matches query)

## jadecli Context Strategy
- CLAUDE.md < 500 lines (essentials only)
- Specialized instructions in skills (18 agent perspectives, 16 commands)
- `/clear` between unrelated tasks
- Subagent delegation for verbose ops (test output, log analysis)
- OTel event pruning in PostToolUse hooks
- Scratchpad pattern for extended architecture sessions
