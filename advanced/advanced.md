# Advance Usage

This section is a work in progress. Topics include the following, but more coming...

- [Sub-agents](#sub-agents)
- [MCP](#mcp)
- [settings.json & Hooks](#settingsjson--hooks)
- [Dev Containers](#development-containers)
- [Git Worktrees](#git-worktrees)

## Hooks & settings.json

### Hooks
Hooks run scripts automatically at specific points in Claude’s workflow. Unlike claude.md instructions which are advisory, hooks are deterministic and guarantee the action happens.

Claude can write hooks for you. Try prompts like “Write a hook that runs eslint after every file edit” or “Write a hook that blocks writes to the migrations folder.” Run /hooks for interactive configuration, or edit [settings.json](#settingsjson) directly.

### settings.json
While  `claude.md` handles instructions and context, `settings.json` gives you programmatic control over Claude’s behavior. This JSON configuration manages permissions, environment variables, and advanced features that go beyond simple instructions.

Claude Code supports multiple settings.json locations:

- **Global**: `~/.claude/settings.json` (applies to all projects)
- **Project**: `.claude/settings.json` (shared with team via version control)
- **Local**: `.claude/settings.local.json` (personal overrides, typically gitignored)

## MCP

With MCP servers, you can allow Claude to connect to external tools and services. For example, you could use MCP to implement features from issue trackers, query databases, analyze monitoring data, integrate designs from Figma, or automate workflows.

> [!Note] 
> Run `claude mcp add` to connect external tools like Jira.
> ```
> claude mcp add --transport http atlassian https://mcp.atlassian.com/v1/mcp
> ```

## Sub-Agents

Subagents are specialized AI assistants that handle specific types of tasks. Each subagent runs in its own context window with a custom system prompt, specific tool access, and independent permissions. When Claude encounters a task that matches a subagent’s description, it delegates to that subagent, which works independently and returns results.

Subagents help you:

- **Preserve context** by keeping exploration and implementation out of your main conversation
- **Enforce constraints** by limiting which tools a subagent can use
- **Reuse configurations** across projects with user-level subagents
- **Specialize behavior** with focused system prompts for specific domains
- **Control costs** by routing tasks to faster, cheaper models like Haiku

Claude uses each subagent’s description to decide when to delegate tasks. When you create a subagent, write a clear description so Claude knows when to use it.

> [!TIP]
> If you need multiple agents working in parallel and communicating with each other, see [agent teams](#agentteams) instead. Subagents work within a single session; agent teams coordinate across separate sessions.

Refer here for pre-configured [awesome-claude-code-subagents](https://github.com/VoltAgent/awesome-claude-code-subagents).

## Agent Teams

> [!INFO]
> Agent teams are disabled by default. Enable them by setting the `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` environment variable to `1`

Agent teams let you coordinate multiple Claude Code instances working together. One session acts as the team lead, coordinating work, assigning tasks, and synthesizing results. Teammates work independently, each in its own context window, and communicate directly with each other.

Unlike [subagents](#sub-agents), which run within a single session and can only report back to the main agent, you can also interact with individual teammates directly without going through the lead.

Agent teams are most effective for tasks where parallel exploration adds real value. The strongest use cases are:

- **Research and review**: multiple teammates can investigate different aspects of a problem simultaneously, then share and challenge each other’s findings
- **New modules or features**: teammates can each own a separate piece without stepping on each other
- **Debugging with competing hypotheses**: teammates test different theories in parallel and converge on the answer faster
- **Cross-layer coordination**: changes that span frontend, backend, and tests, each owned by a different teammate

> [!WARNING] 
> Agent teams add coordination overhead and use significantly more tokens than a single session. They work best when teammates can operate independently. For sequential tasks, same-file edits, or work with many dependencies, a single session or [subagents](https://code.claude.com/docs/en/sub-agents) are more effective.

## Development containers
The reference [devcontainer setup](https://github.com/anthropics/claude-code/tree/main/.devcontainer) and associated [Dockerfile](https://github.com/anthropics/claude-code/blob/main/.devcontainer/Dockerfile) offer a preconfigured development container that you can use as is, or customize for your needs. This devcontainer works with the Visual Studio Code [Dev Containers extension](https://code.visualstudio.com/docs/devcontainers/containers) and similar tools. The container’s enhanced security measures (isolation and firewall rules) allow you to run `claude --dangerously-skip-permissions` to bypass permission prompts for unattended operation.

## Git Worktrees

> [!WARNING]
> This section is still a work in progress

Git worktrees combined with Claude Code create a powerful workflow for parallel development. Instead of juggling multiple branches on a single working directory, you can have multiple isolated environments where different Claude Code instances work simultaneously on separate features without any conflicts.

Git worktrees allow you to check out multiple branches from the same repository into separate directories. Each worktree has its own working directory with isolated files, while sharing the same Git history and reflog.

Think of it this way:
- **Traditional workflow**: One working directory, switch between branches
- **Worktree workflow**: Multiple working directories, each with its own branch