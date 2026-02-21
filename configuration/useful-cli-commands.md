# Useful AWS CLI Commands

### Authentication

```bash
# Login via SSO
aws sso login --profile ai-dev

# Check current AWS identity
aws sts get-caller-identity

# Check which profile is active
echo $AWS_PROFILE
```

### List Models
```bash
# List all available foundation models
aws bedrock list-foundation-models

# List only Claude models
aws bedrock list-foundation-models \
  --query "modelSummaries[?contains(modelId, 'claude')].[modelId,modelName]" \
  --output table

# List Claude models with more details
aws bedrock list-foundation-models \
  --query "modelSummaries[?contains(modelId, 'claude')]" \
  --output table

# List models by provider (Anthropic)
aws bedrock list-foundation-models \
  --by-provider anthropic

# List only models you have access to
aws bedrock list-foundation-models \
  --query "modelSummaries[?modelLifecycle.status=='ACTIVE'].[modelId,modelName]" \
  --output table
```

### Model Access
```bash
# Check model access status
aws bedrock get-foundation-model-availability \
  --model-id anthropic.claude-3-sonnet-20240229-v1:0

# List custom models (fine-tuned)
aws bedrock list-custom-models

# List provisioned model throughputs
aws bedrock list-provisioned-model-throughputs
```

### Invoke Models Directly
```bash
# Invoke Claude 3 Sonnet
aws bedrock-runtime invoke-model \
  --model-id anthropic.claude-3-sonnet-20240229-v1:0 \
  --body '{"anthropic_version":"bedrock-2023-05-31","max_tokens":256,"messages":[{"role":"user","content":"Hello, Sonnet!"}]}' \
  --cli-binary-format raw-in-base64-out \
  output.json

# Invoke Claude 3 Haiku (faster, cheaper)
aws bedrock-runtime invoke-model \
  --model-id anthropic.claude-3-haiku-20240307-v1:0 \
  --body '{"anthropic_version":"bedrock-2023-05-31","max_tokens":256,"messages":[{"role":"user","content":"Hello Haiku!"}]}' \
  --cli-binary-format raw-in-base64-out \
  output.json

# View the response
cat output.json | jq .
```

### Inference Profiles
```bash
# List inference profiles (for cross-region inference)
aws bedrock list-inference-profiles

# List inference profiles for Claude models
aws bedrock list-inference-profiles \
  --query "inferenceProfileSummaries[?contains(inferenceProfileId, 'claude')]" \
  --output table

# Get details on a specific inference profile
aws bedrock get-inference-profile \
  --inference-profile-identifier us.anthropic.claude-3-5-sonnet-20241022-v2:0
```

