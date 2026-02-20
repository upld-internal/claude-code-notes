# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
## Repository Purpose

This is a documentation repository containing personal notes for setting up and using Claude Code with AWS Bedrock, authenticated via SSO. It also includes best practices and setup guides for various Claude Code development workflows. There are no application code, build system, or tests.
## Documentation Structure

- `readme.md` - Main entry point with overview, prerequisites, and basic usage
- `configuration/` - Step-by-step setup guides:
  - `troubleshooting.md` - Common issues and AWS CLI commands
- `advanced.md` - MCP, hooks, dev containers, git worktrees
## Writing Guidelines

When updating documentation:
- Keep shell examples compatible with bash and zsh
- Use the `ai-dev` profile name for consistency in examples
- Reference `us-east-1` as the primary region
- Link between documents using relative paths with URL encoding for spaces
