# OpenCode Installation Guide

> **Repository:** https://github.com/anomalyco/opencode  
> **Description:** The open source AI coding agent  
> **Stars:** 148k+ | **Language:** TypeScript  
> **License:** MIT

---

## Table of Contents

1. [What is OpenCode?](#what-is-opencode)
2. [Quick Install](#quick-install)
3. [Slack MCP Full Setup](#slack-mcp-full-setup)
4. [Configuration](#configuration)
5. [Agents & Usage](#agents--usage)
6. [Troubleshooting](#troubleshooting)

---

## What is OpenCode?

OpenCode is a **100% open source AI coding agent** built for the terminal. It's provider-agnostic and can work with Claude, OpenAI, Google, or local models.

### Key Features

- **Provider Agnostic** - Use Claude, GPT-4, Gemini, or local models
- **Built-in LSP Support** - Language Server Protocol integration
- **TUI Focus** - Terminal User Interface optimized for developers
- **Client/Server Architecture** - Run remotely, drive from mobile
- **Agent System** - Built-in `build` (default) and `plan` (read-only) agents
- **VSCode SDK** - Programmatic integration available

---

## Quick Install

### One-Line Install

```bash
# YOLO install (most popular)
curl -fsSL https://opencode.ai/install | bash
```

### Package Managers

```bash
# npm
npm i -g opencode-ai@latest

# bun/pnpm/yarn
bun add -g opencode-ai@latest
pnpm add -g opencode-ai@latest

# macOS (Homebrew - recommended, always up to date)
brew install anomalyco/tap/opencode

# macOS (official brew formula, updated less)
brew install opencode

# Windows (Scoop)
scoop install opencode

# Windows (Chocolatey)
choco install opencode

# Arch Linux
sudo pacman -S opencode      # Stable
paru -S opencode-bin         # Latest AUR

# Any OS (mise)
mise use -g opencode

# Nix
nix run nixpkgs#opencode
# or for latest dev:
nix run github:anomalyco/opencode
```

### Desktop App (BETA)

```bash
# macOS (Apple Silicon)
brew install --cask opencode-desktop

# Windows (Scoop)
scoop bucket add extras
scoop install extras/opencode-desktop

# Or download directly from:
# https://github.com/anomalyco/opencode/releases
# https://opencode.ai/download
```

### Installation Directory Priority

The install script respects this order:

1. `$OPENCODE_INSTALL_DIR` - Custom directory
2. `$XDG_BIN_DIR` - XDG compliant path
3. `$HOME/bin` - Standard user binary
4. `$HOME/.opencode/bin` - Default fallback

```bash
# Custom install examples
OPENCODE_INSTALL_DIR=/usr/local/bin curl -fsSL https://opencode.ai/install | bash
XDG_BIN_DIR=$HOME/.local/bin curl -fsSL https://opencode.ai/install | bash
```

---

## Slack MCP Full Setup

### Step 1: Create a Slack App

1. Navigate to [api.slack.com/apps](https://api.slack.com/apps)
2. Click **"Create New App"**
3. Select **"From scratch"**
4. App Name: `OpenCode Assistant`
5. Select your workspace
6. Click **"Create App"**

### Step 2: Configure OAuth Scopes

Go to **OAuth & Permissions**:

#### Required Bot Token Scopes

Click **"Add an OAuth Scope"** and add these:

```
app_mentions:read          - Read app mention events
channels:history           - View message history in channels
channels:join              - Join public channels
channels:read              - View basic channel info
chat:write                 - Send messages
chat:write.public          - Send messages to public channels
files:read                 - Read files shared
files:write                - Upload files
groups:history             - View private channel history
groups:read                - View private channel info
groups:write               - Manage private channels
im:history                 - View DM history
im:read                    - View DMs
im:write                   - Send DMs
mpim:history               - View group DM history
mpim:read                  - View group DMs
mpim:write                 - Send group DMs
reactions:read             - View emoji reactions
reactions:write            - Add/remove reactions
users:read                 - View workspace members
users:read.email           - View member email addresses
```

### Step 3: Enable Event Subscriptions

Navigate to **Event Subscriptions**:

1. Toggle **"Enable Events"** to ON
2. For **Socket Mode** (recommended), skip Request URL setup
3. Subscribe to these **Bot Events**:

```
app_mention                - Subscribe to bot mentions
channel_created            - Channel created events
channel_deleted            - Channel deleted events
file_change                - File modified
file_created               - New file uploaded
file_deleted               - File deleted
file_shared                - File shared to channel
file_unshared              - File unshared
group_deleted              - Private channel deleted
member_joined_channel      - User joins channel
member_left_channel        - User leaves channel
message.app_home           - Messages in App Home tab
message.channels           - Messages in public channels
message.groups             - Messages in private channels
message.im                 - Direct messages
message.mpim               - Group direct messages
reaction_added             - Emoji reaction added
reaction_removed           - Emoji reaction removed
```

### Step 4: Enable Socket Mode

Navigate to **Socket Mode**:

1. Toggle **"Socket Mode"** to ON
2. Go to **Basic Information** → **App-Level Tokens**
3. Click **"Generate Token and Scopes"**
4. Token Name: `opencode-socket`
5. Add Scope: `connections:write`
6. Click **"Generate"**
7. **Copy the App-Level Token** (starts with `xapp-`)

### Step 5: Install App to Workspace

1. Go to **OAuth & Permissions** → **OAuth Tokens for Your Workspace**
2. Click **"Install to Workspace"**
3. Review permissions and click **"Allow"**
4. **Copy the Bot User OAuth Token** (starts with `xoxb-`)

### Step 6: Configure OpenCode MCP

Create or edit `~/.opencode/config.json`:

```json
{
  "mcp": {
    "servers": {
      "slack": {
        "command": "npx",
        "args": ["-y", "@modelcontextprotocol/server-slack"],
        "env": {
          "SLACK_BOT_TOKEN": "xoxb-YOUR-BOT-TOKEN-HERE",
          "SLACK_APP_TOKEN": "xapp-YOUR-APP-TOKEN-HERE"
        }
      }
    }
  }
}
```

### Alternative: Direct Slack Integration

For native Slack integration (not MCP):

```bash
# Set environment variables
export SLACK_BOT_TOKEN="xoxb-YOUR-TOKEN"
export SLACK_APP_TOKEN="xapp-YOUR-TOKEN"

# Or configure in OpenCode settings
opencode config set slack.botToken xoxb-xxx
opencode config set slack.appToken xapp-xxx
opencode config set slack.enabled true
```

### Step 7: Invite Bot to Channels

In any Slack channel:

```
/invite @OpenCode
```

### Step 8: Verify Setup

```bash
# Start OpenCode
opencode

# In the TUI, check MCP servers are loaded
/mcp status

# Test Slack integration
# Send a message to the bot in Slack
```

---

## Configuration

### Full Config File (`~/.opencode/config.json`)

```json
{
  "agent": {
    "default": "build",
    "model": "claude-3-5-sonnet-20241022"
  },
  "models": {
    "claude": {
      "apiKey": "sk-ant-xxx",
      "provider": "anthropic"
    },
    "openai": {
      "apiKey": "sk-xxx",
      "provider": "openai"
    }
  },
  "mcp": {
    "servers": {
      "slack": {
        "command": "npx",
        "args": ["-y", "@modelcontextprotocol/server-slack"],
        "env": {
          "SLACK_BOT_TOKEN": "xoxb-xxx",
          "SLACK_APP_TOKEN": "xapp-xxx"
        }
      },
      "github": {
        "command": "npx",
        "args": ["-y", "@modelcontextprotocol/server-github"],
        "env": {
          "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_xxx"
        }
      }
    }
  },
  "slack": {
    "enabled": true,
    "socketMode": true,
    "botToken": "xoxb-xxx",
    "appToken": "xapp-xxx",
    "allowedChannels": ["*"],
    "dmEnabled": true
  }
}
```

### Environment Variables

```bash
# Required
export SLACK_BOT_TOKEN="xoxb-xxx"
export SLACK_APP_TOKEN="xapp-xxx"

# Optional
export OPENCODE_MODEL="claude-3-5-sonnet-20241022"
export OPENCODE_API_KEY="sk-ant-xxx"
export OPENCODE_CONFIG_PATH="~/.opencode/config.json"
```

---

## Agents & Usage

### Built-in Agents

| Agent | Description | Use Case |
|-------|-------------|----------|
| `build` | Full-access agent (default) | Development work, file edits |
| `plan` | Read-only agent | Code exploration, analysis |

### Switching Agents

```bash
# In TUI, press Tab to switch agents
# Or use command:
opencode --agent plan

# Start with specific agent
opencode --agent build
```

### Using Subagents

```bash
# Invoke general subagent for complex tasks
@general search for all async functions in this codebase
```

### CLI Commands

```bash
# Start TUI
opencode

# Start in specific directory
opencode /path/to/project

# Use specific agent
opencode --agent plan

# Run single command
opencode --command "explain this codebase"

# Check version
opencode --version

# Get help
opencode --help
```

### TUI Keybindings

```
Tab          - Switch between build/plan agents
Ctrl+C       - Cancel current operation
Ctrl+D       - Exit
@            - Mention subagent
/            - Slash commands
?            - Help
```

### Slash Commands

```
/new         - Start new conversation
/compact     - Compress conversation context
/model       - Switch LLM model
/config      - Show/edit configuration
/help        - Show help
```

---

## Troubleshooting

### MCP Server Not Loading

```bash
# Check MCP status
opencode
# Then type: /mcp status

# Verify npx is available
which npx
npm install -g npx

# Test MCP server manually
npx -y @modelcontextprotocol/server-slack
```

### Bot Not Responding

1. **Check tokens:**
   ```bash
   echo $SLACK_BOT_TOKEN
   echo $SLACK_APP_TOKEN
   ```

2. **Verify scopes:**
   - Go to api.slack.com/apps → Your App → OAuth & Permissions
   - Ensure all required scopes are added

3. **Reinstall app:**
   - OAuth & Permissions → Reinstall to Workspace

4. **Check Socket Mode:**
   - Basic Information → App-Level Tokens
   - Ensure token has `connections:write` scope

### Rate Limiting

```json
{
  "slack": {
    "rateLimiting": {
      "enabled": true,
      "requestsPerSecond": 1
    }
  }
}
```

### Debug Mode

```bash
# Run with verbose logging
opencode --verbose

# Or set env var
export OPENCODE_DEBUG=1
opencode
```

### Connection Issues

```bash
# Check network
ping slack.com

# Verify firewall isn't blocking WebSocket
nc -zv slack.com 443

# Check proxy settings
env | grep -i proxy
```

### Common Error Messages

| Error | Solution |
|-------|----------|
| `missing_scope` | Add missing scope in OAuth & Permissions |
| `channel_not_found` | Invite bot to channel with `/invite @bot` |
| `not_authed` | Regenerate and update tokens |
| `account_inactive` | Reinstall app to workspace |
| `ratelimited` | Implement rate limiting in config |

### Verification Checklist

```bash
# 1. Check OpenCode installation
opencode --version

# 2. Verify MCP servers
opencode
# /mcp status

# 3. Test Slack connection
# Send DM to bot in Slack

# 4. Check logs
# Logs stored in: ~/.opencode/logs/
tail -f ~/.opencode/logs/latest.log

# 5. Validate config
opencode --check-config
```

---

## Getting Help

- **Documentation:** https://opencode.ai/docs
- **Discord:** https://discord.gg/opencode
- **GitHub Issues:** https://github.com/anomalyco/opencode/issues
- **GitHub Discussions:** https://github.com/anomalyco/opencode/discussions

---

## Advanced Topics

### Custom MCP Server

```typescript
// my-slack-server.ts
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

const server = new Server({
  name: "my-slack-server",
  version: "1.0.0"
});

// Add custom tools...

const transport = new StdioServerTransport();
server.connect(transport);
```

### Multi-Provider Setup

```json
{
  "models": {
    "fast": {
      "provider": "openai",
      "model": "gpt-4o-mini",
      "apiKey": "sk-xxx"
    },
    "smart": {
      "provider": "anthropic",
      "model": "claude-3-5-sonnet-20241022",
      "apiKey": "sk-ant-xxx"
    },
    "local": {
      "provider": "ollama",
      "model": "codellama",
      "baseUrl": "http://localhost:11434"
    }
  }
}
```

---

*Last updated: 2026-04-22*
