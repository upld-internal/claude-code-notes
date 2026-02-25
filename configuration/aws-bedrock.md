# Configure Claude for AWS Bedrock

## Configure Envirionment Variables
The following variables should be set. I add this block to my `~/.bashrc` or `~/.zshrc` so it runs automatically when I open a new terminal.

```bash
# Tell claude to use Bedrock instead of the default
CLAUDE_CODE_USE_BEDROCK=1

# The name of your AWS CLI profile
export AWS_PROFILE=ai-dev

# Claude Code doesn't read from the ~/.aws/config file for this.
export AWS_REGION=us-east-1  # or your preferred region

ANTHROPIC_MODEL=  
'arn:aws:bedrock:us-east-1:860816123468:inference-profile/us.anthropic.claude-sonnet-4-5-20250929-v1:0'

export ANTHROPIC_SMALL_FAST_MODEL='arn:aws:bedrock:us-east-1::foundation-model/anthropic.claude-haiku-4-5-20251001-v1:0'

# Recommended output token settings for Bedrock
export CLAUDE_CODE_MAX_OUTPUT_TOKENS=4096
export MAX_THINKING_TOKENS=1024

# Optional: Disable prompt caching if needed 
export DISABLE_PROMPT_CACHING=1
```