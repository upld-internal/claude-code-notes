# Git Worktrees in Claude Code

Git Worktrees let you run multiple Claude Code sessions simultaneously, each working in an isolated directory on its own branch. This enables true parallel development without branch-switching conflicts.

## Why Use Git Worktrees?

In a standard workflow, you have one working directory and must switch branches to change context. With worktrees, you have multiple working directories — each checked out to a different branch — all sharing the same Git history and remote connections.

**The problem worktrees solve:**
- You're midway through a feature when an urgent bug report comes in
- You want Claude working on one task while you review something else
- You need to run tests on one branch while editing another
- Multiple agents need to make file changes concurrently without conflicts

---

## Starting a Worktree Session

Use the `--worktree` flag (short: `-w`) when launching Claude:

```bash
# Create a named worktree
claude --worktree feature-auth

# Short form
claude -w feature-auth

# Auto-generate a random name
claude --worktree
```

This creates an isolated working directory at `.claude/worktrees/feature-auth/` with a new branch named `worktree-feature-auth` based off your default remote branch. Claude's session runs entirely within that directory.

You can also ask Claude to enter a worktree mid-session:

```
> work in a worktree
> start a worktree for this feature
```

### Running Sessions in Parallel

Open separate terminals for concurrent work:

```bash
# Terminal 1 — work on a new feature
claude --worktree feature-payments

# Terminal 2 — fix an unrelated bug
claude --worktree bugfix-login-timeout

# Terminal 3 — your main branch stays untouched
claude
```

Each session has its own file state. Edits in one worktree never interfere with another.

---

## Worktree Lifecycle

### Creation
- Worktrees are placed at `<repo>/.claude/worktrees/<name>/`
- Branch naming: `worktree-<name>`
- Branch is created from the default remote branch

### Cleanup on Exit

When you exit a worktree session, Claude checks for changes:

| State | Behaviour |
|-------|-----------|
| No changes made | Worktree and branch removed automatically |
| Changes or commits exist | Claude prompts you to keep or remove |

Choosing **keep** preserves the directory and branch for later. Choosing **remove** deletes the worktree and discards any uncommitted work.

### Manual Cleanup

```bash
# List all worktrees
git worktree list

# Remove a specific worktree
git worktree remove .claude/worktrees/feature-auth

# Force remove (discards uncommitted changes)
git worktree remove --force .claude/worktrees/feature-auth
```

### Gitignore

Add this to your `.gitignore` to keep worktree directories out of your repository:

```
.claude/worktrees/
```

---

## Worktrees with Subagents

When Claude spawns subagents (via the Task tool), each agent can run in its own isolated worktree:

```
> Use isolated worktrees for your agents to work in parallel
```

**What happens:**
- Each subagent gets its own `.claude/worktrees/` directory
- Subagents can safely write files without stepping on each other
- Worktrees are cleaned up automatically if the agent makes no changes
- If changes were made, the worktree path and branch are returned so you can review or merge

This is particularly useful for team-based workflows where a lead agent coordinates multiple agents each implementing different parts of a task.

---

## Resuming Worktree Sessions

The session picker (`/resume`) shows sessions from the same Git repository, including those in worktrees. Each entry displays the branch name so you can identify which worktree it belongs to.

---

## Manual Worktree Management

For more control, create worktrees directly with Git before starting Claude:

```bash
# Create a new worktree with a new branch
git worktree add ../project-feature-a -b feature-a

# Create a worktree from an existing branch
git worktree add ../project-bugfix bugfix-123

# Start Claude inside it
cd ../project-feature-a && claude

# Clean up when done
cd .. && git worktree remove project-feature-a
```

> [!NOTE]
> When creating worktrees manually, remember to set up your development environment inside each one — run `npm install`, activate virtual environments, etc.

---

## Hooks for Custom Worktree Behaviour

For projects not using Git (SVN, Perforce, Mercurial), or for customised setup/teardown, you can hook into the worktree lifecycle.

### `WorktreeCreate`

Runs when a worktree is created. Must print the absolute path of the new worktree to stdout.

```json
{
  "hooks": {
    "WorktreeCreate": [{
      "hooks": [{
        "type": "command",
        "command": "bash -c 'NAME=$(jq -r .name); DIR=\"$HOME/.claude/worktrees/$NAME\"; svn checkout https://svn.example.com/repo/trunk \"$DIR\" >&2 && echo \"$DIR\"'"
      }]
    }]
  }
}
```

### `WorktreeRemove`

Runs when a worktree is removed. Use this to perform custom cleanup.

```json
{
  "hooks": {
    "WorktreeRemove": [{
      "hooks": [{
        "type": "command",
        "command": "bash -c 'jq -r .worktree_path | xargs rm -rf'"
      }]
    }]
  }
}
```

> [!NOTE]
> Both hooks only support `type: "command"`. `WorktreeRemove` cannot block the removal — it is for cleanup only.

---

## When to Use Worktrees

| Scenario | Recommendation |
|----------|----------------|
| Urgent bug while mid-feature | Worktree for the bug, keep your feature branch untouched |
| Two independent features in parallel | One worktree per feature |
| Running tests while editing code | Worktree for testing, main directory for edits |
| Subagents making concurrent file changes | Pass `isolation: "worktree"` to each agent |
| Reviewing a colleague's branch | Manual worktree checkout, no context-switching required |
| Non-Git VCS (SVN, Perforce) | `WorktreeCreate` / `WorktreeRemove` hooks |

---

## Summary

- `claude --worktree <name>` starts an isolated session on a new branch
- Worktrees live at `.claude/worktrees/<name>/` and are cleaned up on exit
- Use multiple terminals with different `--worktree` names for parallel Claude sessions
- Subagents support worktree isolation for concurrent file changes without conflicts
- Customise creation and teardown with `WorktreeCreate` and `WorktreeRemove` hooks
