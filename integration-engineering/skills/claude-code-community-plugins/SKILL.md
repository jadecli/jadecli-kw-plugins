---
name: claude-code-community-plugins
description: Create, install, distribute, and manage Claude Code plugins. Marketplace setup, plugin structure, skills, agents, hooks, MCP servers, and community distribution.
argument-hint: "<create|install|marketplace|migrate|submit>"
---

# Claude Code Community Plugins

Create, distribute, and manage plugins for Claude Code and Cowork.

## Install Plugins

```bash
# Add a marketplace
claude plugin marketplace add owner/repo

# Install a plugin
claude plugin install plugin-name@marketplace-name

# Scoped install
claude plugin install plugin-name@marketplace-name --scope project

# List installed
claude plugin list

# Update marketplaces
claude plugin marketplace update marketplace-name
```

## Create a Plugin

### 1. Directory Structure

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json          # Manifest (only this goes here)
├── skills/                   # Agent skills (SKILL.md files)
│   └── my-skill/
│       └── SKILL.md
├── commands/                 # Slash commands (markdown files)
├── agents/                   # Custom agent definitions
├── hooks/
│   └── hooks.json            # Event handlers
├── .mcp.json                 # MCP server configs
├── .lsp.json                 # LSP server configs
├── settings.json             # Default settings
└── README.md
```

### 2. Plugin Manifest

```json
// .claude-plugin/plugin.json
{
  "name": "my-plugin",
  "description": "What it does",
  "version": "1.0.0",
  "author": { "name": "Your Name" }
}
```

### 3. Add a Skill

```markdown
<!-- skills/review/SKILL.md -->
---
name: review
description: Reviews code for best practices. Use when reviewing PRs.
---

When reviewing code, check for:
1. Code organization
2. Error handling
3. Security concerns
4. Test coverage
```

### 4. Test Locally

```bash
claude --plugin-dir ./my-plugin
# Then try:
/my-plugin:review
```

Use `/reload-plugins` to pick up changes without restarting.

## Create a Marketplace

### Marketplace Manifest

```json
// .claude-plugin/marketplace.json
{
  "name": "my-marketplace",
  "owner": { "name": "My Org" },
  "plugins": [
    {
      "name": "my-plugin",
      "source": "./my-plugin",
      "description": "What it does"
    }
  ]
}
```

### External Plugin Sources

```json
{
  "name": "external-plugin",
  "source": {
    "source": "url",
    "url": "https://github.com/org/plugin-repo.git",
    "sha": "abc123"
  }
}
```

## Plugin Components

### Skills (model-invoked)

Claude auto-uses based on task context. `$ARGUMENTS` captures user input.

```markdown
---
name: deploy
description: Deploy the application
argument-hint: "<environment>"
---
Deploy to $ARGUMENTS environment...
```

### Commands (user-invoked)

Slash commands with `/plugin-name:command-name`.

### Agents

Custom agent definitions in `agents/` directory.

### Hooks

```json
// hooks/hooks.json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [{ "type": "command", "command": "npm run lint:fix" }]
      }
    ]
  }
}
```

### MCP Servers

```json
// .mcp.json
{
  "mcpServers": {
    "my-server": {
      "command": "node",
      "args": ["server.js"]
    }
  }
}
```

### Default Settings

```json
// settings.json
{ "agent": "security-reviewer" }
```

## Migrate from .claude/ to Plugin

1. Create `.claude-plugin/plugin.json`
2. Copy `commands/`, `agents/`, `skills/` to plugin root
3. Move hooks from `settings.json` to `hooks/hooks.json`
4. Test with `--plugin-dir`

## Submit to Official Marketplace

- Claude.ai: https://claude.ai/settings/plugins/submit
- Console: https://platform.claude.com/plugins/submit

## Standalone vs Plugin

| Standalone (.claude/) | Plugin |
|---|---|
| `/hello` | `/plugin-name:hello` |
| One project only | Shareable via marketplace |
| Quick experiments | Versioned releases |

## Official Marketplace Plugins

Browse at: `claude plugin marketplace add anthropics/knowledge-work-plugins`

Categories: data, engineering, sales, finance, legal, marketing, customer-support, product-management, design, operations, human-resources, productivity, enterprise-search.

## Best Practices

- Start standalone in `.claude/`, convert to plugin when ready to share
- Use semantic versioning in plugin.json
- Include README.md with install and usage instructions
- Test with `--plugin-dir` before publishing
- Namespace prevents conflicts between plugins
