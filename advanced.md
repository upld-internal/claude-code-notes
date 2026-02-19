# Advance Usage

This section is a work in progress. This will probably be broken out into separate documents. Topics include the following, but more coming...
- [MCP](#mcp)
- [settings.json & Hooks](#settingsjson--hooks)
- [Dev Containers](#development-containers)
- [Git Worktrees](#git-worktrees)

## MCP
With MCP servers, you can allow Claude to connect to external services. For example, you could use MCP to implement features from issue trackers, query databases, analyze monitoring data, integrate designs from Figma, or automate workflows.

> **Note**: Run `claude mcp` add to connect external tools like Jira or AWS.

## settings.json & Hooks
### settings.json
While  `claude.md` handles instructions and context, `settings.json` gives you programmatic control over Claude’s behavior. This JSON configuration manages permissions, environment variables, and advanced features that go beyond simple instructions.

Claude Code supports multiple settings.json locations:

- **Global**: `~/.claude/settings.json` (applies to all projects)
- **Project**: `.claude/settings.json` (shared with team via version control)
- **Local**: `.claude/settings.local.json` (personal overrides, typically gitignored)

### Hooks
Hooks run scripts automatically at specific points in Claude’s workflow. Unlike CLAUDE.md instructions which are advisory, hooks are deterministic and guarantee the action happens.

Claude can write hooks for you. Try prompts like “Write a hook that runs eslint after every file edit” or “Write a hook that blocks writes to the migrations folder.” Run /hooks for interactive configuration, or edit `.claude/settings.json` directly.

## Development containers
The reference [devcontainer setup](https://github.com/anthropics/claude-code/tree/main/.devcontainer) and associated [Dockerfile](https://github.com/anthropics/claude-code/blob/main/.devcontainer/Dockerfile) offer a preconfigured development container that you can use as is, or customize for your needs. This devcontainer works with the Visual Studio Code [Dev Containers extension](https://code.visualstudio.com/docs/devcontainers/containers) and similar tools. The container’s enhanced security measures (isolation and firewall rules) allow you to run `claude --dangerously-skip-permissions` to bypass permission prompts for unattended operation.

## Git Worktrees
Git worktrees combined with Claude Code create a powerful workflow for parallel development. Instead of juggling multiple branches on a single working directory, you can have multiple isolated environments where different Claude Code instances work simultaneously on separate features without any conflicts.

Git worktrees allow you to check out multiple branches from the same repository into separate directories. Each worktree has its own working directory with isolated files, while sharing the same Git history and reflog.

Think of it this way:
- **Traditional workflow**: One working directory, switch between branches
- **Worktree workflow**: Multiple working directories, each with its own branch