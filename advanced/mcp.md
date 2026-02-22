# MCP (Model Context Protocol) in Claude Code

MCP (Model Context Protocol) is an open standard that allows Claude Code to connect to external tools, services, and data sources. It enables Claude to interact with systems like Jira, GitHub, databases, and more through a standardized interface.

## What is MCP?

MCP servers act as bridges between Claude and external services. When you configure an MCP server, Claude gains access to new tools that can:

- Query external APIs and databases
- Create, read, and update records in external systems
- Access data that lives outside your codebase
- Perform actions in connected services

Think of MCP as a plugin system that extends Claude's capabilities beyond your local filesystem.

---

## Why Use MCP?

**Connect to External Services** - Integrate with tools your team already uses - Jira, GitHub, Slack, Sentry, databases, and more.

**Automate Workflows** - Let Claude create tickets, query issues, update records, and perform actions across systems.

**Access Remote Data** - Query databases, search documentation, and pull information from APIs directly in your Claude session.

**Team Consistency** - Share MCP configurations via version control so everyone has the same integrations.

---

## Adding MCP Servers

### HTTP/Remote Servers

Most MCP servers are remote services accessed via HTTP:

```bash
claude mcp add --transport http <name> <url>
```

**Examples:**
```bash
# Atlassian Jira
claude mcp add --transport http jira https://mcp.atlassian.com/v2/mcp
```

### Local/Stdio Servers

Some MCP servers run locally as processes:

```bash
claude mcp add --transport stdio <name> -- <command> [args]
```

**Examples:**
```bash
# Filesystem server
claude mcp add --transport stdio filesystem -- npx -y @modelcontextprotocol/server-filesystem /path/to/dir

# PostgreSQL server
claude mcp add --transport stdio postgres -- npx -y @modelcontextprotocol/server-postgres postgresql://localhost/mydb
```

### Managing Servers

```bash
# List all configured servers
claude mcp list

# Get details about a server
claude mcp get <name>

# Remove a server
claude mcp remove <name>

# Check status and authenticate (in Claude Code)
/mcp
```

---

## Configuration Scopes

MCP servers can be configured at different levels:

| Scope | Location | Visibility | Use Case |
|-------|----------|------------|----------|
| **Local** (default) | `~/.claude.json` | Just you, current project | Personal, experimental |
| **Project** | `.mcp.json` in project root | Team via Git | Shared team tools |
| **User** | `~/.claude.json` | All your projects | Personal utilities |

### Setting Scope

```bash
# Project scope - shared with team
claude mcp add --scope project --transport http github https://api.githubcopilot.com/mcp/

# User scope - available everywhere
claude mcp add --scope user --transport http sentry https://mcp.sentry.dev/mcp

# Local scope (default)
claude mcp add --transport http jira https://mcp.atlassian.com/v2/mcp
```

### Project Configuration File

For team sharing, create `.mcp.json` in your project root:

```json
{
  "mcpServers": {
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/"
    },
    "jira": {
      "type": "http",
      "url": "https://mcp.atlassian.com/v2/mcp"
    }
  }
}
```

> [!NOTE]
> Claude prompts for approval before using servers from `.mcp.json` the first time. Reset with `claude mcp reset-project-choices`.

---

## Authentication

Most MCP servers require authentication via OAuth 2.0.

### Interactive Authentication

```bash
# 1. Add the server
claude mcp add --transport http jira https://mcp.atlassian.com/v2/mcp

# 2. In Claude Code, authenticate
/mcp
# Select the server and choose "Authenticate"
# Follow the browser login flow
```

### Pre-configured OAuth

For automated setups or custom servers:

```bash
claude mcp add --transport http \
  --client-id your-client-id \
  --client-secret \
  --callback-port 8080 \
  my-server https://mcp.example.com/mcp
```

### Managing Authentication

```bash
# In Claude Code
/mcp
# Options: Authenticate, Clear authentication, View servers
```

---

## Example: Atlassian Jira MCP

The Atlassian MCP server enables Claude to interact with Jira issues and Confluence pages.

### Setup

```bash
# Add the Jira MCP server
claude mcp add --transport http jira https://mcp.atlassian.com/v2/mcp

# Authenticate in Claude Code
/mcp
# Select "jira" and authenticate via browser
```

### For Team Setup

Add to `.mcp.json`:

```json
{
  "mcpServers": {
    "jira": {
      "type": "http",
      "url": "https://mcp.atlassian.com/v2/mcp"
    }
  }
}
```

### Using Jira in Claude Code

Once configured, you can ask Claude to work with Jira directly:

**Reading feature documentation:**
```
Read the requirements from Jira issue ABC-12345 and summarize the acceptance criteria
```

**Querying issues:**
```
What are the highest priority bugs assigned to me in Jira?
Show me all open tasks in the current sprint
```

**Creating and updating:**
```
Create a Jira issue for the authentication bug we just found
Update ABC-12345 to mark it as in progress
```

**Linking work:**
```
Implement the feature described in ABC-12345 and create a PR linking to it
```

### Practical Workflow Example

Here's a complete workflow using Jira MCP to implement a feature:

```
1. "Fetch the details of Jira issue ABC-12345"
   → Claude retrieves the issue title, description, and acceptance criteria

2. "What files are likely involved based on this feature request?"
   → Claude analyzes your codebase and suggests relevant files

3. "Implement the feature according to the Jira requirements"
   → Claude writes the code based on the Jira ticket specs

4. "Create a PR with a link to the Jira issue"
   → Claude creates a PR with ABC-12345 referenced in the description
```

---

## MCP vs Skills

Understanding when to use MCP servers versus Skills is important for effective Claude Code usage.

### Key Differences

| Aspect | MCP Servers | Skills |
|--------|-------------|--------|
| **Purpose** | External tool/API integrations | Internal workflows & instructions |
| **Execution** | Remote service or local process | Within Claude Code session |
| **Setup** | Medium-High (separate service) | Low (just a `.md` file) |
| **Provides** | New tools for Claude to use | New commands and workflows |
| **State** | Can maintain persistent state | Stateless per invocation |
| **Latency** | 100-2000ms (network) | Instant |

### When to Use MCP

Use MCP servers when you need to:

- **Connect to external services** - Jira, GitHub, Slack, databases
- **Access remote data** - Query APIs, search external documentation
- **Perform side effects** - Create tickets, send messages, update systems
- **Share integrations** - Team needs access to the same tools

**Example:**
```bash
# Add Jira MCP to query and create issues
claude mcp add --transport http jira https://mcp.atlassian.com/v2/mcp
```

### When to Use Skills

Use Skills when you need to:

- **Teach Claude your conventions** - Code style, API patterns, deployment steps
- **Create custom commands** - `/deploy`, `/review-pr`, `/explain-code`
- **Guide specific workflows** - Step-by-step instructions for complex tasks
- **Add context knowledge** - Domain-specific information without external calls

**Example:**
```yaml
---
name: deploy
description: Deploy the application to production
---

Deploy to production:
1. Run the test suite
2. Build the Docker image
3. Push to container registry
4. Update the deployment
```

### Decision Guide

```
Need to connect to an external service/API?
├─ YES → MCP Server
└─ NO ↓

Need to add new tools/capabilities?
├─ YES → MCP Server
└─ NO ↓

Need to teach Claude your conventions?
├─ YES → Skill
└─ NO ↓

Need a reusable workflow/command?
├─ YES → Skill
└─ NO → Neither (just ask Claude directly)
```

### Using Both Together

MCP and Skills complement each other well:

```yaml
---
name: implement-feature
description: Implement a feature from Jira requirements
---

Implement the Jira feature:

1. Fetch the issue details from Jira using the MCP tool
2. Extract requirements and acceptance criteria
3. Analyze existing code structure
4. Implement the feature following our coding standards
5. Write tests for the new functionality
6. Create a PR linking to the Jira issue
```

This skill uses MCP (to fetch Jira data) while providing workflow guidance (the step-by-step process).

---

## Security Considerations

### Project Server Approval

Claude prompts for approval before using MCP servers from `.mcp.json`. This prevents malicious servers from being added without your knowledge.

### Permission Control

Control MCP access in `.claude/settings.json`:

```json
{
  "permissions": {
    "allow": [
      "mcp__github",
      "mcp__jira__*"
    ],
    "deny": [
      "mcp__untrusted__*"
    ]
  }
}
```

### Allowlists and Denylists

For stricter control:

```json
{
  "allowedMcpServers": [
    { "serverName": "github" },
    { "serverName": "jira" },
    { "serverUrl": "https://mcp.company.com/*" }
  ],
  "deniedMcpServers": [
    { "serverUrl": "https://*.untrusted.com/*" }
  ]
}
```

### Credential Storage

- OAuth credentials are stored in the system keychain (macOS) or secure storage
- Client secrets are NOT stored in `.mcp.json`
- Environment variables can be used for tokens: `${JIRA_API_TOKEN}`

---

## Common MCP Servers

| Server | Purpose | URL |
|--------|---------|-----|
| **GitHub** | PRs, issues, repos | `https://api.githubcopilot.com/mcp/` |
| **Atlassian** | Jira, Confluence | `https://mcp.atlassian.com/v2/mcp` |
| **Sentry** | Error monitoring | `https://mcp.sentry.dev/mcp` |
| **PostgreSQL** | Database queries | Local stdio server |
| **Filesystem** | Extended file access | Local stdio server |

Find more at the [MCP Registry](https://github.com/modelcontextprotocol/servers).

---

## Quick Reference

| Task | Command |
|------|---------|
| Add HTTP server | `claude mcp add --transport http name url` |
| Add stdio server | `claude mcp add --transport stdio name -- command args` |
| List servers | `claude mcp list` |
| Get details | `claude mcp get name` |
| Remove server | `claude mcp remove name` |
| Authenticate | `/mcp` (in Claude Code) |
| Project scope | `--scope project` |
| User scope | `--scope user` |
| Reset approvals | `claude mcp reset-project-choices` |

