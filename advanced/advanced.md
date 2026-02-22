# Advance Usage

This section is a work in progress. Topics include the following, but more coming...

- [Sub-agents](#sub-agents)
- [MCP](#mcp)
- [settings.json & Hooks](#settingsjson--hooks)
- [Dev Containers](#development-containers)
- [Git Worktrees](#git-worktrees)

## settings.json

While  `claude.md` handles instructions and context, `settings.json` gives you programmatic control over Claude’s behavior. This JSON configuration manages permissions, environment variables, and advanced features that go beyond simple instructions.

See [configuring settings.json] for more.

## MCP

MCP (Model Context Protocol) is an open standard that allows Claude Code to connect to external tools, services, and data sources. It enables Claude to interact with systems like Jira, GitHub, databases, and more through a standardized interface.

See [MCP](mcp.md) for more info.

## Sub-Agents

Subagents are specialized AI assistants that handle specific types of tasks. Each subagent runs in its own context window with a custom system prompt, specific tool access, and independent permissions. When Claude encounters a task that matches a subagent’s description, it delegates to that subagent, which works independently and returns results.

See [Sub-Agents](sub-agents.md) for more info and check out this collection of [awesome claude code subagents](https://github.com/VoltAgent/awesome-claude-code-subagents).

## Agent Teams

Agent teams let you coordinate multiple Claude Code instances working together. One session acts as the team lead, coordinating work, assigning tasks, and synthesizing results. Teammates work independently, each in its own context window, and communicate directly with each other.

Unlike subagents, which run within a single session and can only report back to the main agent, you can also interact with individual teammates directly without going through the lead. 

See [Multi Agent Teams](multi-agent-teams.md) for more info.

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