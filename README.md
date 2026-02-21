# Claude Code with AWS Bedrock and SSO

These are my personal notes for configuring Claude Code for use with AWS Bedrock. I've also included basic usage and advanced use cases.

I will try to update periodically, but if you notice any errors, please feel free to submit a PR, [file an issue](https://github.com/upld-internal/claude-code-notes/issues/new), or [contact me](mailto:bripley@uplandsoftware.com) so I can update.

## Overview

[Claude Code](https://code.claude.com/docs/en/overview) is Anthropic's official CLI tool for interacting with Claude. When using it with AWS Bedrock, Claude runs within our AWS infrastructure, giving us control over data residency and security. 

[AWS Bedrock](https://aws.amazon.com/bedrock/) provides a single interface for foundation models from Anthropic, Meta, Mistral, Stability AI, and Amazon, allowing for text, image, and chat-based applications without managing infrastructure.

## Configuration Steps

> [!WARNING]
> Before proceeding, you should be able to login to AWS console in a browser via SSO and see your account number and a role that has access to Bedrock. Ask your AWS administrator if you're not sure where to find these.

#### Prerequisites

- AWS account number
- Name of AWS Role with Bedrock access
- SSO Start URL  
- [AWS CLI v2](https://aws.amazon.com/cli/) 

Follow the steps below to configure your AWS profile, enable SSO authentication for AWS CLI, install Claude Code, and configure Claude to use Bedrock.

1. [AWS Profile and SSO Authentication](configuration/aws-sso.md)
2. [Install Claude Code](configuration/claude-code.md)
3. [Set Environment Variables for AWS Bedrock](configuration/aws-bedrock.md)

Once configured, you can start Claude Code by changing to your project folder and running:

```
claude
```

![[configuration/claude.png]]

See [troubleshooting](configuration/troubleshooting.md) for more help with common errors when using claude with bedrock.

## Useful CLI Commands

Reference [useful AWS CLI commands](configuration/useful-cli-commands).

## Basic Usage

See [basic usage](basic.md) for common commands and pro tips for getting started with Claude Code including:

- Common commands
- Config files (claude.md & skills)
- AI-assisted development best practices 

## Advanced Usage

See [advanced usage](advanced.md) for more advanced workflows with claude code including:

- MCP
- Hooks & settings.json
- Sub-Agents
- Agent Teams
- Dev Containers
- Git Worktrees
























