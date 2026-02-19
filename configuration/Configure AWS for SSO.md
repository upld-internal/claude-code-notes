# Configure AWS for SSO
To set up a custom profile for AWS Single Sign-On (SSO) in the AWS CLI, you use the interactive `aws configure sso` command and provide a specific profile name when prompted. 

### Prerequisites

- AWS Console Access
- Duo (SSO) Configured
- [AWS CLI v2](https://aws.amazon.com/cli/) installed
- SSO Start URL ([https://upland.awsapps.com/start](https://upland.awsapps.com/start))  
- SSO Region (`us-east-1`)

## Configure SSO

In your terminal, run the aws sso configuration command:

`aws configure sso`

#### Provide SSO details

1. **SSO session name:** You will be prompted to create a name for the SSO session (e.g., `my-sso-session`). This is used to group configuration variables.
2. **SSO start URL:** Enter the full URL for your AWS access portal ([https://upland.awsapps.com/start]([`https://upland.awsapps.com/start)).
3. **SSO region:** Enter the region where your IAM Identity Center is located (`us-east-1`).
4. **SSO registration scopes:** You can typically leave this blank and press **Enter**.

#### Authenticate via browser

The CLI will automatically open your default web browser to an AWS authorization page. Follow the prompts in the browser to sign in to your AWS access portal and grant the CLI permission to access your accounts.

#### Select account and role

Back in your terminal, the CLI will list the AWS accounts and permission sets (roles) available to you. Use your arrow keys to select the desired **AWS account** and the appropriate **role** (e.g., `ReadOnly` or `FullAccess`).

#### Specify profile details

1. **CLI default client Region:** Enter your desired default AWS region for this specific profile (e.g., `us-east-1`).
2. **CLI default output format:** Enter your preferred output format (e.g., `json`, `text`, `table`).
3. **CLI profile name:** This is where you set your custom profile name (e.g., `ai-dev`). If left blank, the CLI suggests a default name. 

## Using A Custom Profile

After configuration, your `~/.aws/config` file will be updated with the profile details. You can then use the custom profile with AWS CLI commands by specifying the `--profile` option: 

To check if the profile is working correctly, run:

```
aws sts get-caller-identity --profile ai-dev
```

You can also set the `AWS_PROFILE` environment variable to use the profile for all subsequent commands in a terminal session without the `--profile` flag. 

After configuring, my `~/.aws/config file` looks like this:

```
[default]
region = us-east-2

[profile ai-dev]
sso_session = my-sso-session
sso_account_id = 905418058350
sso_role_name = USS_AI_DEV_Bedrock_Access
region = us-east-1

[sso-session my-sso-session]
sso_start_url = https://upland.awsapps.com/start
sso_region = us-east-1
sso_registration_scopes = sso:account:access
```

## Signing In

Your credentials are temporary and will expire after a set duration (typically 8-12 hours). To refresh them, run the `aws sso login` command: 

```
aws sso login --profile ai-dev
```

This will open your browser again for re-authentication.



