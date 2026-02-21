# Troubleshooting

### SSO Session Expired

Your credentials are temporary and will expire after a set duration (typically 8-12 hours). If you see authentication errors, run the `aws sso login` command to refresh your SSO session:

```bash
aws sso login --profile ai-dev
```

This will open your browser again for re-authentication.

### Model Access Denied

Verify that:
1. Your AWS account has Bedrock model access enabled
2. Your IAM role/permission set includes Bedrock invoke permissions
3. You're using a region where Bedrock and Claude are available

### Region Availability

Claude models on Bedrock are available in specific regions. Check the [AWS Bedrock documentation](https://docs.aws.amazon.com/bedrock/latest/userguide/models-regions.html) for current availability. As of this writing, we are only using Bedrock in `us-east-1` (N. Virginia)

