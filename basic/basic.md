# Basic Usage

## Common Commands

#### Standard shell commands while inside claude
Use `!` to run non-claude terminal commands within the claude code cli. 
```
! ls -al
```
#### Switch models 
Use the `/model` slash command to switch models
```
/model opus
/model sonnet
/model haiku
```
#### Clear context
Use `/clear` slash command to clear context
```
/clear
```
#### Use plan mode
Press `Shift+Tab` to cycle to the "plan mode" indicator (⏸) in the terminal, then enter your complex request; Claude will analyze and create a detailed, step-by-step plan without making changes, which you can review, edit (`Ctrl+G`), and approve before execution, separating thinking from doing for safer, more complex refactors. 


## Adding Context With Config Files

### claude.md
`claude.md` is a special file that Claude reads at the start of every conversation. Include Bash commands, code style, and workflow rules. This gives Claude persistent project-wide context it can’t infer from code alone.

There’s no required format for claude.md files, but keep it short and human-readable. Include context about project structure, code style, instructions to lint/build/test/run, project conventions, constraints, library versions, etc. 

The claude.md is loaded every session, so only include things that apply broadly. For domain knowledge or workflows that are only relevant sometimes, use [Skills](#skillsmd) instead. Claude loads them on demand without bloating every conversation.

You can place claude.md files in several locations:
- **Home folder**:
    -  `~/.claude/claude.md`: Applies to all Claude sessions
- **Project root**:
    - name it `./claude.md` and check into git to share with your team
    - or name it `claude.local.md` and .gitignore it to use it for personal preferences
- **Parent directories**:
    - Useful for monorepos where both `root/claude.md` and `root/foo/CLAUDE.md` are pulled in automatically
- **Child directories**: 
    - Claude pulls in `child claude.md` files on demand when working with files in those directories

> [!TIP]
> Run `/init` to generate a starter claude.md file based on your current project structure, then refine over time.

### Skills.md
Skills teach Claude how to complete specific tasks in a repeatable way. Skills are folders of instructions, scripts, and resources that Claude loads dynamically to improve performance on specialized tasks, or you can invoke one directly with /skill-name. 

Skills are typically added to a hidden `.claude/skills` skills directory in the root of your project. For example `.claude/skills/code-review/skill.md` 

Check out [anthropics/skills](https://github.com/anthropics/skills/tree/main/skills) for some great examples of pre-made skills or learn more about [how to create your own custom skills](https://support.claude.com/en/articles/12512198-how-to-create-custom-skills).


## Best Practices
Claude Code is an agentic coding environment. Unlike a chatbot that answers questions and waits, Claude Code can read your files, run commands, make changes, and autonomously work through problems while you watch, redirect, or step away entirely.

This changes how you work. Instead of writing code yourself and asking Claude to review it, you describe what you want and Claude figures out how to build it. Claude explores, plans, and implements. Find Anthropic's updated recommended best practices guide in the [anthropic docs](https://code.claude.com/docs/en/best-practices).
### Provide context in your prompts
When prompting, you need to be specific about what you are asking AI to do and use explicit instructions. This may seem obvious, but it's where most prompts fail. The more information you provide, the more accurate the generated response will be. This often requires that you think more deeply about what you are asking before you type your prompt. 

Start with context, the way you’d brief a new teammate. If you don’t share constraints, library versions, project rules, and intended behavior, the model will happily invent them for you.

You can provide rich data to Claude in several ways:
- Reference specific files with `@` instead of describing where code lives. Claude reads the file before responding.
- Paste images directly. Copy/paste or drag and drop images into the prompt.
- Give URLs for documentation and API references. Use `/permissions` to allowlist frequently-used domains.
- Pipe in data by running `cat error.log | claude` to send file contents directly.
- Let Claude fetch what it needs. Tell Claude to pull context itself using Bash commands, MCP tools, skills, or by reading files.

### Explore → Plan → Code

Letting Claude jump straight to coding can produce code that solves the wrong problem. Use Plan Mode to separate exploration from execution.

1. Before you ask AI to write or plan anything, first ask it to **explore**. '*How is authentication currently implemented in this codebase?*’ Help it to get the lay of the land and understand the design patterns in place.

2. Then **plan**. '*Given what you found, outline an approach for adding password reset functionality.*’  Review the plan, adjust it, make sure it makes sense.

3. Only then do you **code**. Now AI has context, you've validated the approach, and you're executing a plan rather than hoping for the best.

This feels slower, but it prevents the 'delete everything and start over' cycle that happens when you rush straight into code. Also, it forces you to think carefully about the system design before you implement. This is often the most difficult part of software development and is also where the most value is added.

| Step        | What You Do                                | Why It Matters                          |
| ----------- | ------------------------------------------ | --------------------------------------- |
| **Explore** | Understand codebase structure and patterns | Prevents reinventing existing solutions |
| **Plan**    | Create detailed approach before coding     | Catches issues before they're built     |
| **Code**    | Execute the plan with AI assistance        | Focused implementation, fewer rewrites  |

The recommended workflow has four phases:

1. **Explore**: Press `Shift+Tab` to enter Plan Mode. Claude reads files and answers questions without making changes.
2. **Plan**: Ask Claude to create a detailed implementation plan. Optionally, press `Ctrl+G` to edit the plan before Claude proceeds.
3. **Implement**: Press `Shift+Tab` to switch back to Normal Mode and let Claude code, verifying against its plan.
4. **Commit**: Ask Claude to commit with a descriptive message and create a pull request.

### Give Claude a way to verify its work

AI tools perform dramatically better when they can verify their own work, like run tests, compare screenshots, and validate outputs. Without clear success criteria, it might produce something that looks right but actually doesn’t work. You become the only feedback loop, and every mistake requires your attention, making you the bottleneck. Invest in making your verification rock-solid.

Don't just ask for code — ask for proof that the code works. AI can run tests, check types, and verify its own output, but only if you ask. Include tests, screenshots, or expected outputs so Claude can check itself. Every implementation request should include verification! 

 **Verification Techniques**
- Write tests alongside implementation
- Run existing tests after changes
- Check for linting errors
- Verify TypeScript/type errors
- Confirm the code compiles

> [!TIP] 
> Providing Claude a way to verify it's output is the single highest-leverage thing you can do!
