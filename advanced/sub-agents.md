# Sub-Agents in Claude Code

Sub-agents are specialized AI assistants that handle specific tasks within Claude Code. They operate with their own isolated context, allowing you to delegate work without cluttering your main conversation.

## What Are Sub-Agents?

Sub-agents are lightweight, focused workers that:
- Run in their own isolated context window
- Have customizable tool access and permissions
- Can use different models (Haiku, Sonnet, Opus)
- Return only a summary to your main conversation

This isolation is valuable because verbose operations (like running tests) stay contained rather than consuming your main context.

---

## Why Use Sub-Agents?

### Isolate Verbose Output
Test runs, log analysis, and documentation generation produce thousands of lines. Sub-agents process this internally and return only what matters.

```
Use a subagent to run the test suite and report only failing tests
```

### Parallel Research
Run multiple investigations simultaneously without context bloat.

```
Research the authentication and payment modules in parallel using separate subagents
```

### Cost Optimization
Use cheaper models (Haiku) for exploration tasks while keeping your main conversation on Sonnet or Opus.

### Tool Restrictions
Limit what a sub-agent can do - useful for read-only analysis or security-restricted operations.

---

## How to Use Sub-Agents

### Automatic Delegation

Simply describe the task and Claude decides when to delegate:

```
Use a subagent to analyze the error logs and summarize the issues
```

### Explicit Request

Request a specific sub-agent by name:

```
Use the code-reviewer agent to check this module
Have the test-runner subagent investigate the failing tests
```

### Built-In Sub-Agents

Claude Code includes these sub-agents:

| Sub-Agent | Model | Purpose |
|-----------|-------|---------|
| **Explore** | Haiku | Fast, read-only codebase exploration |
| **Plan** | Inherits | Research during plan mode |
| **General-purpose** | Inherits | Complex multi-step tasks |

---

## Creating Custom Sub-Agents

### Using the Interactive Interface

```bash
/agents
```

This opens a guided setup to create, edit, or delete sub-agents.

### Manual File Creation

Create files in `.claude/agents/` (project) or `~/.claude/agents/` (user):

**`.claude/agents/code-reviewer.md`**:
```markdown
---
name: code-reviewer
description: Reviews code for quality and security issues. Use proactively after code changes.
tools: Read, Grep, Glob, Bash
model: sonnet
---

You are a senior code reviewer. When invoked:
1. Run git diff to see recent changes
2. Focus on modified files
3. Report issues by priority: Critical, Warnings, Suggestions

Review for:
- Code clarity and readability
- Proper error handling
- No exposed secrets or API keys
- Input validation
```

### Configuration Options

| Field | Description |
|-------|-------------|
| `name` | Unique identifier (lowercase, hyphens) |
| `description` | When Claude should delegate to this sub-agent |
| `tools` | Allowed tools (inherits all if omitted) |
| `model` | `haiku`, `sonnet`, `opus`, or `inherit` |
| `permissionMode` | `default`, `acceptEdits`, `dontAsk`, `bypassPermissions` |
| `maxTurns` | Maximum agentic turns before stopping |
| `memory` | `user`, `project`, or `local` for persistent memory |

---

## Model Selection

| Model | Cost | Speed | Best For |
|-------|------|-------|----------|
| `haiku` | Lowest | Fastest | Exploration, simple tasks |
| `sonnet` | Medium | Balanced | Most custom agents |
| `opus` | Highest | Slowest | Complex reasoning |
| `inherit` | Parent's | Parent's | Default behavior |

---

## Common Patterns

### Read-Only Reviewer
```yaml
---
name: security-scanner
description: Scans code for security vulnerabilities
tools: Read, Grep, Glob
model: haiku
---
```

### Full-Access Debugger
```yaml
---
name: debugger
description: Investigates and fixes errors and test failures
tools: Read, Edit, Bash, Grep, Glob
model: sonnet
---
```

### Test Runner
```yaml
---
name: test-runner
description: Runs tests and reports failures concisely
tools: Read, Bash, Grep
model: haiku
---
```

---

## Token and Cost Considerations

Sub-agents can reduce token usage by:
- **Isolating verbose output** - Only summaries return to main conversation
- **Using cheaper models** - Haiku for exploration is significantly cheaper
- **Bounding context** - Each sub-agent starts fresh without parent context

**When sub-agents save tokens:**
- High-volume operations (test runs, log analysis)
- Parallel research tasks
- Read-only exploration

**When they don't help much:**
- Small, quick tasks
- Tasks requiring main conversation context
- Multiple sub-agents returning detailed results

---

## Best Practices

1. **One sub-agent, one purpose** - Keep them focused
2. **Write detailed descriptions** - Helps Claude know when to delegate
3. **Limit tools appropriately** - Read-only agents don't need Edit
4. **Use Haiku for exploration** - Fast and cheap
5. **Check project sub-agents into version control** - Share with your team

---

## Limitations

- **Cannot nest** - Sub-agents cannot spawn other sub-agents
- **Independent context** - They don't see your main conversation history
- **Background restrictions** - No MCP tools in background tasks

---

## Sub-Agents vs Multi Agent Teams

For simple, focused tasks, sub-agents are the right choice. For complex collaborative work, consider [Multi Agent Teams](multi-agent-teams.md).

| Aspect | Sub-Agents | Multi Agent Teams |
|--------|-----------|-------------------|
| **Communication** | Returns summary only | Agents talk to each other |
| **Coordination** | None | Shared task list |
| **Setup** | Simple | Requires configuration |
| **Cost** | Lower | Higher (multiple contexts) |
| **Best for** | Focused tasks | Parallel collaboration |

**Use Sub-Agents when:**
- Task is self-contained
- You need verbose output isolated
- Running parallel but independent research
- Cost optimization is a priority

**Use Multi Agent Teams when:**
- Agents need to coordinate
- Complex problem requires multiple perspectives
- Work spans multiple modules needing discussion
- You want shared task management

See the [Multi Agent Teams Guide](multi-agent-teams.md) for detailed information on that feature.
