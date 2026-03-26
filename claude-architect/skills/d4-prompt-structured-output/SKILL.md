---
name: d4-prompt-structured-output
description: "Cert Domain 4 (20%): Explicit criteria prompts, few-shot examples, JSON schema enforcement via tool_use, validation/retry loops, batch processing, multi-pass review."
argument-hint: "<task-statement-number>"
---

# D4: Prompt Engineering & Structured Output (20%)

## 4.1 Explicit Criteria
- "Flag when claimed behavior contradicts actual code" > "check that comments are accurate"
- "Be conservative" and "only report high-confidence" fail — use specific categorical criteria
- High false-positive rates in one category undermine trust in all categories
- Define explicit severity criteria with concrete code examples per level

## 4.2 Few-Shot Prompting
- Most effective technique for consistent, actionable output
- Show reasoning for ambiguous scenarios (why action A over B)
- Demonstrate specific desired output format (location, issue, severity, fix)
- Distinguish acceptable patterns from genuine issues to reduce false positives
- Handle varied document structures (inline citations vs bibliographies)
- Show correct extraction from documents with varied/null formats

## 4.3 Structured Output via tool_use
- `tool_use` with JSON schemas = guaranteed schema-compliant output (no syntax errors)
- `tool_choice: "auto"` — model may return text; `"any"` — must call a tool; forced — specific tool
- Strict schemas eliminate syntax errors but NOT semantic errors (values in wrong fields)
- Required vs optional fields: make nullable when source may not contain the data
- Add `"unclear"` / `"other"` enum values with detail fields for extensibility
- Format normalization rules alongside strict output schemas

## 4.4 Validation & Retry Loops
- Retry-with-error-feedback: append specific validation errors to prompt on retry
- Effective for: format mismatches, structural errors (2-3 attempts resolves most)
- Ineffective for: missing information (source doesn't contain the data — fail fast)
- `detected_pattern` field in findings enables systematic false-positive analysis
- Self-correction: extract `calculated_total` + `stated_total`, flag discrepancy

## 4.5 Batch Processing
- Message Batches API: 50% cost savings, up to 24-hour processing window
- Appropriate for: overnight reports, weekly audits, nightly test generation
- NOT for: pre-merge checks, blocking workflows
- No multi-turn tool calling in batch (can't execute tools mid-request)
- `custom_id` for correlating request/response pairs
- Prompt refinement on sample set before batch-processing at volume

## 4.6 Multi-Pass Review Architecture
- Self-review limitation: model retains reasoning context, less likely to question own decisions
- Independent review instances (no prior context) catch more subtle issues
- Multi-pass: per-file local analysis + cross-file integration pass (avoids attention dilution)
- Verification passes with model self-reported confidence for calibrated routing

## jadecli Application
- Staff review team: 4.6 multi-pass (6 independent specialist reviewers)
- Code review plugin: 4.1 explicit severity criteria (REVIEW.md)
- Telemetry extraction: 4.3 structured output with Zod schemas
- WBR generation: 4.5 batch for weekly aggregation
