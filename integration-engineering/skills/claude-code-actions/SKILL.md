---
name: claude-code-actions
description: Set up Claude Code GitHub Actions for automated PR creation, code review, issue triage, and CI/CD automation. @claude mentions in PRs and issues.
argument-hint: "<setup|workflow|bedrock|vertex|troubleshoot>"
---

# Claude Code GitHub Actions

Run Claude Code in your GitHub CI pipeline. @claude mentions in PRs/issues trigger automated code changes, reviews, and implementations.

## Quick Setup

```bash
# In Claude Code terminal:
/install-github-app
```

Or manually:
1. Install GitHub App: https://github.com/apps/claude
2. Add `ANTHROPIC_API_KEY` to repo secrets
3. Copy workflow from examples/claude.yml

## Basic Workflow

```yaml
name: Claude Code
on:
  issue_comment:
    types: [created]
  pull_request_review_comment:
    types: [created]
jobs:
  claude:
    runs-on: ubuntu-latest
    steps:
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

## Action Parameters

| Parameter | Description | Required |
|-----------|-------------|----------|
| `prompt` | Instructions (plain text or skill name) | No |
| `claude_args` | CLI arguments passed to Claude Code | No |
| `anthropic_api_key` | Claude API key | Yes* |
| `github_token` | GitHub token for API access | No |
| `trigger_phrase` | Custom trigger (default: "@claude") | No |
| `use_bedrock` | Use AWS Bedrock | No |
| `use_vertex` | Use Google Vertex AI | No |

## Common Patterns

### Code Review on PR

```yaml
- uses: anthropics/claude-code-action@v1
  with:
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
    prompt: "Review this PR for code quality, correctness, and security"
    claude_args: "--max-turns 5"
```

### Daily Report (Scheduled)

```yaml
on:
  schedule:
    - cron: "0 9 * * *"
steps:
  - uses: anthropics/claude-code-action@v1
    with:
      anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
      prompt: "Generate a summary of yesterday's commits and open issues"
```

### With Skills

```yaml
- uses: anthropics/claude-code-action@v1
  with:
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
    prompt: "/code-review"
```

### Claude Args Examples

```yaml
claude_args: "--max-turns 5 --model claude-sonnet-4-6 --allowedTools Read,Edit,Bash"
```

## @claude Trigger Examples

In issue or PR comments:
```
@claude implement this feature based on the issue description
@claude fix the TypeError in the user dashboard component
@claude how should I implement auth for this endpoint?
```

## Cloud Providers

### AWS Bedrock

```yaml
- uses: aws-actions/configure-aws-credentials@v4
  with:
    role-to-assume: ${{ secrets.AWS_ROLE_TO_ASSUME }}
    aws-region: us-west-2
- uses: anthropics/claude-code-action@v1
  with:
    use_bedrock: "true"
    claude_args: '--model us.anthropic.claude-sonnet-4-6'
```

### Google Vertex AI

```yaml
- uses: google-github-actions/auth@v2
  with:
    workload_identity_provider: ${{ secrets.GCP_WORKLOAD_IDENTITY_PROVIDER }}
    service_account: ${{ secrets.GCP_SERVICE_ACCOUNT }}
- uses: anthropics/claude-code-action@v1
  with:
    use_vertex: "true"
    claude_args: '--model claude-sonnet-4@20250514'
```

## GitHub App Permissions

Contents: Read & Write, Issues: Read & Write, Pull requests: Read & Write.

## Best Practices

- Create CLAUDE.md for project-specific guidelines
- Use `--max-turns` to control costs
- Set workflow timeouts to prevent runaway jobs
- Use GitHub concurrency controls for parallel runs
