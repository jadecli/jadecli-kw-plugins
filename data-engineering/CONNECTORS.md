# Connectors

Works standalone. Supercharged with these MCP servers:

| Connector | MCP Server | Used By |
|-----------|-----------|---------|
| Supabase | `@supabase/mcp-server-supabase` | `supabase-cube`, `/cube-model` |
| GitHub | `@modelcontextprotocol/server-github` | `package-manager`, `/packages` |
| Cube.js | `@cubejs-backend/server` REST API | `supabase-cube`, `/cube-model` |

## Placeholder Tokens

| Placeholder | Meaning |
|-------------|---------|
| `{{SUPABASE_PROJECT_REF}}` | Supabase project reference ID |
| `{{SUPABASE_DB_URL}}` | Supabase PostgreSQL connection string |
| `{{CUBE_API_URL}}` | Cube.js API endpoint |
| `{{GITHUB_OWNER}}` | GitHub org (default: `jadecli`) |
