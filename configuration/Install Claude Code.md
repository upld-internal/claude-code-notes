# Claude Code Installation

### Native Install (recommended)

macOS, Linux, WSL:
```
curl -fsSL https://claude.ai/install.sh | bash
```

Windows PowerShell:
```
irm https://claude.ai/install.ps1 | iex
```

Windows CMD:
```
curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd
```
**Note**: Native installations automatically update in the background to keep you on the latest version.


### Install via homebrew (mac only)
```
brew install --cask claude-code
```
**Note**: Homebrew installations do not auto-update. Run `brew upgrade claude-code` periodically to get the latest features and security fixes.

## Verify installation
Verify the installation by checking the version:
```
claude --version
```