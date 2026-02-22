# Settings Configuration in Claude Code

Claude Code uses `settings.json` files to configure behaviour, permissions, environment variables, and hooks. Understanding the different configuration locations and their purposes helps you customize Claude Code for personal use and team collaboration.

## How Settings Work

Settings control various aspects of Claude Code:

- **Permissions** - What tools Claude can use and what actions require approval
- **Environment variables** - Variables available to Claude and spawned processes
- **Model selection** - Default model to use
- **Hooks** - Custom scripts that run at specific lifecycle events
- **MCP configuration** - External tool integrations

Settings are loaded from multiple locations with a defined precedence order, allowing personal preferences to coexist with team standards.

---

## Configuration Locations

### User Settings: `~/.claude/settings.json`

**Scope:** All projects on your machine
**Visibility:** Private to you
**Use case:** Personal defaults and preferences

```json
{
  "model": "sonnet",
  "env": {
    "EDITOR": "code"
  },
  "permissions": {
    "allow": [
      "Read",
      "Glob",
      "Grep"
    ]
  }
}
```

**When to use:**
- Default model preference
- Personal environment variables
- Global permission defaults
- Hooks you want everywhere (like logging)

---

### Project Settings: `.claude/settings.json`

**Scope:** Single project
**Visibility:** Shared with team via Git
**Use case:** Team standards and project-specific configuration

```json
{
  "model": "sonnet",
  "env": {
    "NODE_ENV": "development"
  },
  "permissions": {
    "allow": [
      "Bash(npm run *)",
      "Bash(npm test)",
      "Bash(git status)",
      "Bash(git diff *)"
    ],
    "deny": [
      "Bash(rm -rf *)",
      "Read(.env)"
    ]
  }
}
```

**When to use:**
- Team-wide permission rules
- Project-specific environment variables
- Shared hooks for code quality
- Standard tooling configurations

> [!TIP]
> Commit `.claude/settings.json` to version control so all team members share the same configuration.

---

### Local Settings: `.claude/settings.local.json`

**Scope:** Single project
**Visibility:** Private (add to `.gitignore`)
**Use case:** Personal overrides for a specific project

```json
{
  "model": "opus",
  "env": {
    "DEBUG": "true"
  },
  "permissions": {
    "allow": [
      "Bash(npm run dev)"
    ]
  }
}
```

**When to use:**
- Override team settings for your workflow
- Personal debugging configuration
- Testing new hooks before sharing
- Credentials or paths specific to your machine

> [!IMPORTANT]
> Always add `.claude/settings.local.json` to your `.gitignore` to avoid committing personal overrides.

---

### Enterprise/Managed Settings

For organizations, administrators can deploy managed settings that cannot be overridden:

| Location | Platform |
|----------|----------|
| `/Library/Application Support/ClaudeCode/managed-settings.json` | macOS |
| `/etc/claude-code/managed-settings.json` | Linux |
| `C:\ProgramData\ClaudeCode\managed-settings.json` | Windows |

Managed settings enforce organizational policies like:
- Restricting allowed MCP servers
- Blocking dangerous commands
- Enforcing specific hooks

---

## Precedence Order

When the same setting exists in multiple files, higher precedence wins:

```
1. Enterprise/Managed settings (highest - cannot be overridden)
2. Local settings (.claude/settings.local.json)
3. Project settings (.claude/settings.json)
4. User settings (~/.claude/settings.json) (lowest)
```

**Example conflict:**
```json
// User settings (~/.claude/settings.json)
{ "model": "haiku" }

// Project settings (.claude/settings.json)
{ "model": "sonnet" }

// Local settings (.claude/settings.local.json)
{ "model": "opus" }
```

**Result:** `opus` is used (local settings win)

> [!NOTE]
> Settings at the same level do NOT merge - higher precedence completely replaces lower for that setting.

---

## Common Configuration Options

### Model Selection

```json
{
  "model": "sonnet"
}
```

Options: `haiku`, `sonnet`, `opus`

### Environment Variables

```json
{
  "env": {
    "NODE_ENV": "development",
    "DEBUG": "true",
    "PATH": "$PATH:/custom/bin",
    "API_KEY": "${API_KEY}"
  }
}
```

Supports variable expansion with `$VAR` or `${VAR}` syntax.

### Permissions

Control what Claude can do without asking:

```json
{
  "permissions": {
    "allow": [
      "Bash(npm run *)",
      "Bash(git status)",
      "Read(src/**)",
      "WebFetch(domain:github.com)"
    ],
    "deny": [
      "Bash(rm -rf *)",
      "Read(.env)",
      "Read(./secrets/**)"
    ]
  }
}
```

**Permission patterns:**
- `Bash(npm run *)` - Allow Bash only for specific commands
- `Read(src/**)` - Allow reading files matching glob pattern
- `WebFetch(domain:github.com)` - Restrict web fetches to specific domains
- `mcp__servername` - Control MCP server access

---

## Hooks

Hooks are custom scripts or prompts that run at specific lifecycle events. They enable automation, validation, and custom workflows.

### Available Hook Events

| Event | When It Fires | Common Use Cases |
|-------|---------------|------------------|
| `PreToolUse` | Before a tool executes | Validate inputs, block dangerous commands |
| `PostToolUse` | After a tool completes | Log activity, run tests after edits |
| `Stop` | When Claude finishes a response | Verify work is complete, check tests pass |
| `SessionStart` | When a session begins | Set up environment, load context |
| `ConfigChange` | When configuration changes | Audit tracking |

### Hook Types

| Type | Description | Use When |
|------|-------------|----------|
| `command` | Run a shell script | Deterministic checks, external tools |
| `prompt` | Send prompt to Claude | Simple judgment calls |
| `agent` | Spawn an agent for verification | Complex analysis, multi-step checks |

### Hook Configuration Structure

```json
{
  "hooks": {
    "EventName": [{
      "matcher": "ToolName|Pattern",
      "hooks": [{
        "type": "command|prompt|agent",
        "command": "script-to-run.sh",
        "timeout": 60
      }]
    }]
  }
}
```

### Example: Block Dangerous Commands

Prevent accidental destructive operations:

**`.claude/settings.json`:**
```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "Bash",
      "hooks": [{
        "type": "command",
        "command": ".claude/hooks/validate-bash.sh"
      }]
    }]
  }
}
```

**`.claude/hooks/validate-bash.sh`:**
```bash
#!/bin/bash
INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command')

# Block dangerous patterns
if echo "$COMMAND" | grep -qE '(rm -rf /|drop table|DELETE FROM)'; then
  jq -n '{
    hookSpecificOutput: {
      hookEventName: "PreToolUse",
      permissionDecision: "deny",
      permissionDecisionReason: "Dangerous command blocked by hook"
    }
  }'
  exit 0
fi

exit 0
```

### Example: Auto-Format After Edits

Run prettier after file changes:

```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Write|Edit",
      "hooks": [{
        "type": "command",
        "command": ".claude/hooks/format-file.sh"
      }]
    }]
  }
}
```

**`.claude/hooks/format-file.sh`:**
```bash
#!/bin/bash
INPUT=$(cat)
FILE_PATH=$(echo "$INPUT" | jq -r '.tool_input.file_path // empty')

if [[ "$FILE_PATH" == *.ts || "$FILE_PATH" == *.js || "$FILE_PATH" == *.json ]]; then
  npx prettier --write "$FILE_PATH" 2>/dev/null
fi

exit 0
```

### Example: Run Tests After Code Changes

```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Write|Edit",
      "hooks": [{
        "type": "command",
        "command": ".claude/hooks/run-tests.sh",
        "async": true,
        "timeout": 300
      }]
    }]
  }
}
```

### Example: Verify Completion Before Stopping

Use an agent hook to check work is complete:

```json
{
  "hooks": {
    "Stop": [{
      "hooks": [{
        "type": "agent",
        "prompt": "Verify that all unit tests pass by running: npm test",
        "timeout": 120
      }]
    }]
  }
}
```

### Example: Log All Commands

Audit trail of Bash commands:

```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Bash",
      "hooks": [{
        "type": "command",
        "command": "jq -r '.tool_input.command' >> ~/.claude/command-log.txt"
      }]
    }]
  }
}
```

### Managing Hooks

Use `/hooks` in Claude Code to:
- View all configured hooks and their sources
- Add new hooks interactively
- Delete existing hooks
- Temporarily disable all hooks

---

## When and Why to Use Hooks

### Use Hooks For

**Code Quality:**
- Run linters after edits
- Format code automatically
- Validate test coverage

**Security:**
- Block dangerous commands
- Audit sensitive operations
- Validate file paths

**Workflow Automation:**
- Run tests after changes
- Deploy after successful builds
- Update documentation

**Compliance:**
- Log all operations
- Enforce naming conventions
- Track configuration changes

### Hook Best Practices

1. **Make scripts executable** - `chmod +x .claude/hooks/*.sh`
2. **Use absolute paths** - Reference `$CLAUDE_PROJECT_DIR` for reliability
3. **Quote variables** - Use `"$VAR"` to prevent injection
4. **Handle errors gracefully** - Don't break Claude's workflow on hook failures
5. **Test locally first** - Use `.claude/settings.local.json` before sharing
6. **Keep hooks fast** - Use `async: true` for slow operations

---

## Complete Example Configuration

**`.claude/settings.json`** (shared with team):

```json
{
  "model": "sonnet",
  "env": {
    "NODE_ENV": "development"
  },
  "permissions": {
    "allow": [
      "Bash(npm run *)",
      "Bash(npm test)",
      "Bash(git status)",
      "Bash(git diff *)",
      "Bash(git log *)",
      "Read",
      "Glob",
      "Grep"
    ],
    "deny": [
      "Bash(rm -rf *)",
      "Read(.env)",
      "Read(.env.*)"
    ]
  },
  "hooks": {
    "PostToolUse": [{
      "matcher": "Write|Edit",
      "hooks": [{
        "type": "command",
        "command": ".claude/hooks/format-file.sh"
      }]
    }],
    "PreToolUse": [{
      "matcher": "Bash",
      "hooks": [{
        "type": "command",
        "command": ".claude/hooks/validate-bash.sh"
      }]
    }]
  }
}
```

**`.claude/settings.local.json`** (personal overrides):

```json
{
  "model": "opus",
  "env": {
    "DEBUG": "true"
  }
}
```

---

## Troubleshooting

### Hook Not Firing
- Check matcher pattern (case-sensitive)
- Verify correct event name
- Run `/hooks` to see configured hooks

### Permission Denied on Hook Script
```bash
chmod +x .claude/hooks/*.sh
```

### Hook Timeout
Increase the timeout value or use `async: true`:
```json
{
  "timeout": 300,
  "async": true
}
```

### Settings Not Taking Effect
- Restart Claude Code after editing files directly
- Use `/hooks` to review current configuration
- Check precedence order if multiple files exist

---

## Summary

| File | Scope | Shared | Purpose |
|------|-------|--------|---------|
| `~/.claude/settings.json` | All projects | No | Personal defaults |
| `.claude/settings.json` | Project | Yes (Git) | Team standards |
| `.claude/settings.local.json` | Project | No | Personal overrides |
| Managed settings | Organization | Yes | Enterprise policies |

Use hooks to automate workflows, enforce standards, and add custom validation. Start with simple command hooks and progress to agent hooks for complex verification needs.
