# Multi Agent Teams in Claude Code

Multi Agent Teams is an experimental feature in Claude Code that enables multiple Claude instances to work together on complex tasks with coordination, communication, and shared context. This guide covers setup, usage, comparison with sub-agents, and best practices.

## Overview

Multi Agent Teams allows you to spawn multiple Claude "teammates" that can:
- Work on different parts of a project simultaneously
- Share a task list for coordination
- Communicate with each other
- Operate with independent context windows

This is particularly useful for large-scale refactors, parallel code reviews, and complex features spanning multiple layers of an application.

> [!WARNING]
> **Token Consumption Warning**: Multi Agent Teams can significantly increase token usage and costs. Each teammate maintains its own context and consumes tokens independently. Sessions with 4-5 teammates can cost $15-30+, while 6+ teammates may exceed $30 per session. Plan accordingly and monitor usage.

---

## Enabling the Feature

Multi Agent Teams is currently an experimental feature that must be explicitly enabled.

### Environment Variable

```bash
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
```

### Project Settings

Add to `.claude/settings.json`:

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

---

## Teammate Modes

Claude Code supports different modes for how teammates run:

| Mode | Description | Best For |
|------|-------------|----------|
| `in-process` | Teammates run in the same terminal process | Simple setups, quick collaboration |
| `tmux` | Teammates run in separate tmux panes | Power users, complex debugging |
| `iterm2` | Teammates run in iTerm2 tabs/splits | macOS users with iTerm2 |

### Setting the Mode

Via command line:
```bash
claude --teammate-mode tmux
```

Via settings:
```json
{
  "teammateMode": "tmux"
}
```

---

## Usage

### Spawning Teammates

Request teammates in natural language:

```
Create an agent team to work on this authentication refactor. Spawn three teammates:
1. Backend developer - handle the API changes
2. Test engineer - write comprehensive tests
3. Security reviewer - audit for vulnerabilities

Use Sonnet for all teammates.
```

You can also specify model preferences:
```
Spawn a teammate using Haiku to handle the linting checks
```

### Interacting with Teammates

**Keyboard Shortcuts:**
- `Shift+Down` / `Shift+Up` - Cycle through teammates
- `Esc` - Interrupt current teammate
- `Ctrl+T` - View shared task list

**Give instructions to specific teammates:**
```
[Cycle to backend developer]
Add rate limiting to the login endpoint with a 5 request/minute limit
```

### Cleaning Up

When work is complete:
```
All teammates have finished. Clean up the team.
```

---

## Configuration for Agent Teams

### Recommended Project Setup

Create a well-documented `CLAUDE.md` so teammates get proper context:

```markdown
# Project Overview
This is a microservices platform with frontend, backend, and database layers.

# Architecture
- **Frontend**: React application in `/frontend`
- **Backend**: Node.js API in `/backend`
- **Database**: PostgreSQL migrations in `/db`

# Development Setup
npm install
npm run dev

# Code Style
- Use TypeScript with strict mode
- Follow ESLint and Prettier configurations
- Write tests for all new features
```

### Permissions Configuration

Set permissions in `.claude/settings.json` to reduce interruptions:

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  },
  "teammateMode": "in-process",
  "permissions": {
    "allow": [
      "Bash(npm run *)",
      "Bash(git commit *)",
      "Bash(git push *)",
      "Read",
      "Edit"
    ]
  }
}
```

---

## Multi Agent Teams vs Sub-Agents

Understanding when to use each approach is crucial for effective Claude Code usage.

### Feature Comparison

| Feature | Sub-Agents | Agent Teams |
|---------|-----------|-------------|
| **Independent context** | Yes | Yes |
| **Direct inter-agent communication** | No | Yes |
| **Shared task list** | No | Yes |
| **Automatic coordination** | No | Yes |
| **Setup complexity** | Low | Medium |
| **Token cost** | Lower | Higher |
| **Best for** | Quick focused tasks | Collaborative parallel work |

### When to Use Sub-Agents

Sub-agents are lightweight, focused workers best for:

- **Quick exploratory research** - Reading documentation, exploring codebases
- **Read-only analysis** - Code reviews, security scans
- **High-volume verbose operations** - Test runs, log analysis
- **Sequential verification tasks** - Running checks one after another
- **Simple delegated tasks** - "Research this library and summarize findings"

**Example:**
```
Use a subagent to analyze the authentication module and report any security issues
```

### When to Use Agent Teams

Agent teams excel at:

- **Code reviews requiring discussion** - Different reviewers need to share findings
- **Large refactors across multiple modules** - Different teams own different parts
- **Bug investigation with competing hypotheses** - Multiple theories explored in parallel
- **Features spanning multiple layers** - Frontend, backend, and database work in coordination
- **Complex debugging** - Multiple agents investigating different aspects

**Example:**
```
Create an agent team with a frontend developer and backend developer to implement
the new checkout flow. They need to coordinate on the API contract.
```

### Decision Matrix

| Scenario | Recommendation | Reason |
|----------|---------------|--------|
| Quick code review | Sub-Agent | Fast, low overhead |
| Parallel code review with discussion | Agent Team | Reviewers can communicate |
| Run tests & report failures | Sub-Agent | Simple summary result |
| Debug complex issue | Agent Team | Multiple theories in parallel |
| Explore new library | Sub-Agent | Research only, no collaboration |
| Large refactor (3+ modules) | Agent Team | Different teams own modules |
| File migration (100+ files) | Sub-Agents in loop | Sequential batch processing |
| Full-stack feature | Agent Team | Parallel development with coordination |

---

## Terminal Integration

### tmux Integration

tmux mode provides powerful multi-pane visibility for agent teams.

**Setup:**
```bash
# Install tmux if needed
brew install tmux  # macOS
sudo apt install tmux  # Ubuntu/Debian

# Start Claude with tmux mode
claude --teammate-mode tmux
```

**Benefits:**
- See all teammates working simultaneously in separate panes
- Use native tmux commands for navigation (`Ctrl+B` + arrow keys)
- Easily scroll through individual teammate output
- Persist sessions across terminal disconnects

**tmux Commands:**
- `Ctrl+B` + `%` - Split pane horizontally
- `Ctrl+B` + `"` - Split pane vertically
- `Ctrl+B` + arrow keys - Navigate between panes
- `Ctrl+B` + `z` - Toggle pane zoom

### iTerm2 Integration

For macOS users, iTerm2 mode provides native tab/split support.

**Setup:**
```bash
# Ensure iTerm2 is installed and set as default terminal
claude --teammate-mode iterm2
```

**Benefits:**
- Native macOS integration
- Use iTerm2 keyboard shortcuts
- Better rendering and performance
- Integration with iTerm2 profiles

**iTerm2 Shortcuts:**
- `Cmd+D` - Split pane horizontally
- `Cmd+Shift+D` - Split pane vertically
- `Cmd+[` / `Cmd+]` - Navigate between panes
- `Cmd+Option+Arrow` - Navigate by direction

### Comparison of Terminal Tools

| Feature | in-process | tmux | iTerm2 |
|---------|-----------|------|--------|
| Installation required | None | tmux | iTerm2 |
| Multi-pane visibility | No | Yes | Yes |
| Session persistence | No | Yes | No |
| Platform support | All | All | macOS only |
| Best for | Simple tasks | Complex debugging | macOS power users |

---

## Token Consumption and Cost Management

### Understanding Costs

Agent teams significantly increase token usage because:

1. **Independent contexts** - Each teammate maintains its own full context
2. **Communication overhead** - Messages between teammates consume tokens
3. **Parallel processing** - Multiple agents reading/processing simultaneously
4. **Shared task list** - Task list state synchronized across all teammates

### Estimated Costs

| Team Size | Typical Session Cost |
|-----------|---------------------|
| 2-3 teammates | $5-15 |
| 4-5 teammates | $15-30 |
| 6+ teammates | $30+ |

### Optimization Tips

1. **Minimize initial context** - Keep spawn prompts concise
   ```
   Good: "Review src/auth/ for security issues"
   Bad: "Review src/auth/ for security issues. The app was built in 2018..."
   ```

2. **Use Haiku for lightweight tasks** - Cheaper model for simple work
   ```
   Spawn a Haiku teammate to run linting checks
   ```

3. **Limit teammate count** - Only spawn what you need

4. **Enable tool search for large MCP configs**
   ```bash
   export ENABLE_TOOL_SEARCH=auto:5
   ```

5. **Use hooks for preprocessing** - Filter data before Claude sees it
   ```json
   {
     "hooks": {
       "PreToolUse": [{
         "matcher": "Bash",
         "hooks": [{
           "type": "command",
           "command": "~/.claude/hooks/filter-output.sh"
         }]
       }]
     }
   }
   ```

---

## Pros and Cons

### Pros

- **True parallelism** - Multiple agents work simultaneously on different parts
- **Inter-agent communication** - Teammates can coordinate and share findings
- **Shared task management** - Built-in task list keeps everyone aligned
- **Specialized roles** - Assign specific responsibilities to each teammate
- **Complex problem solving** - Multiple perspectives on difficult issues
- **Faster completion** - Large tasks finish quicker with parallel work

### Cons

- **Higher token costs** - Significant increase in API usage
- **Experimental status** - Feature may change or have bugs
- **Complexity** - More moving parts to manage
- **Coordination overhead** - Sometimes teammates may duplicate work
- **Context limitations** - Each teammate has independent (not shared) memory
- **Setup required** - Needs environment variables and configuration
- **Not available in headless mode** - Cannot use in CI/CD pipelines

---

## Real-World Example Workflow

**Scenario**: Refactor authentication while adding OAuth support.

### Step 1: Enable and Start
```bash
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
claude --teammate-mode tmux
```

### Step 2: Request the Team
```
Create an agent team to refactor our authentication system:

1. Backend developer - refactor session code and add Google OAuth
2. Test engineer - write tests for all auth flows
3. Documentation writer - update docs and add examples
4. Security reviewer - audit changes for vulnerabilities

Use Sonnet for all. Security reviewer should approve before implementation begins.
```

### Step 3: Monitor Progress
```
Press Shift+Down to cycle through teammates
Check what each is working on
Press Esc to interrupt and give feedback
Use Ctrl+T to view shared task list
```

### Step 4: Provide Direction
```
[Cycle to backend developer]
Add rate limiting to the login endpoint

[Cycle to test engineer]
Test the OAuth callback handler edge cases

[Cycle to security reviewer]
Review token storage - we're using httpOnly cookies
```

### Step 5: Clean Up
```
All teammates have finished. Clean up the team.
```

---

## Troubleshooting

### Common Issues

**"Agent teams feature not available"**
- Ensure `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` is set
- Restart Claude Code after setting the environment variable

**Teammates not spawning**
- Check that you have sufficient API quota
- Verify the teammate mode is supported on your platform

**tmux panes not appearing**
- Ensure tmux is installed: `which tmux`
- Check tmux is running: `tmux list-sessions`

**High costs**
- Reduce teammate count
- Use Haiku for simple tasks
- Monitor token usage in session

---

## Known Limitations

- Not available in headless/non-interactive mode
- Requires experimental feature flag
- Limited support in Claude Desktop app (CLI preferred)
- Teammates cannot directly share context/memory
- Maximum useful team size around 5-6 before coordination degrades

