# Configuring Claude Code with AWS Bedrock and SSO

Before proceeding, you should be able to login to AWS console in a browser via SSO and see your account number and a role that has access to Bedrock. Ask your AWS administrator if you're not sure where to find these.
## Prerequisites

- AWS account number
- Name of AWS Role with Bedrock access
- SSO Start URL  
- [AWS CLI v2](https://aws.amazon.com/cli/) 
## Configuration Steps

You will need to configure several things:

1. [AWS Profile and SSO Authentication](aws-sso.md)
2. [Install Claude Code](claude-code.md)
3. [Set Environment Variables for AWS Bedrock](aws-bedrock.md)

Once configured, you can start Claude Code by changing to your project folder and running:

```
claude
```

![[claude.png]]