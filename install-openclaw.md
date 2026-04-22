# OpenClaw Installation Guide

> **Repository:** https://github.com/openclaw/openclaw  
> **Description:** Your own personal AI assistant. Any OS. Any Platform. The lobster way. 🦞  
> **Stars:** 362k+ | **Language:** TypeScript/JavaScript  
> **License:** MIT

---

## Table of Contents

1. [What is OpenClaw?](#what-is-openclaw)
2. [Quick Install](#quick-install)
3. [Slack MCP Full Setup](#slack-mcp-full-setup)
4. [Configuration](#configuration)
5. [Troubleshooting](#troubleshooting)

---

## What is OpenClaw?

OpenClaw is a **personal AI assistant** you run on your own devices. It answers you on the channels you already use including Slack, Discord, Telegram, WhatsApp, and 20+ other messaging platforms.

### Key Features

- **Multi-channel inbox** - Unified messaging across all platforms
- **Local-first Gateway** - Control plane for sessions, channels, tools, and events
- **Voice Wake + Talk Mode** - Voice interaction on macOS/iOS/Android
- **Live Canvas** - Agent-driven visual workspace
- **Skills System** - Onboarding-driven setup with bundled/managed/workspace skills
- **Sandboxing** - Docker/SSH/OpenShell backends for security
- **DM Pairing Security** - Unknown senders require pairing code approval

---

## Quick Install

### Requirements

- **Runtime:** Node 24 (recommended) or Node 22.16+
- **OS:** macOS, Linux, Windows (WSL2 strongly recommended)
- **Package Manager:** npm, pnpm, or bun

### Installation Commands

```bash
# Via npm
npm install -g openclaw@latest

# Via pnpm (recommended for builds from source)
pnpm add -g openclaw@latest

# Via bun
bun add -g openclaw@latest
```

### First-Time Setup

```bash
# Interactive onboarding (recommended)
openclaw onboard --install-daemon

# The onboard wizard will guide you through:
# - Gateway setup
# - Workspace configuration
# - Channel connections (Slack, Discord, etc.)
# - Skills setup
```

### Start the Gateway

```bash
# Start gateway with verbose logging
openclaw gateway --port 18789 --verbose

# Or run in background (if daemon installed)
openclaw gateway start
```

---

## Slack MCP Full Setup

### Step 1: Create a Slack App

1. Go to [api.slack.com/apps](https://api.slack.com/apps)
2. Click **"Create New App"**
3. Choose **"From scratch"**
4. Name your app (e.g., "OpenClaw Assistant")
5. Select your workspace
6. Click **"Create App"**

### Step 2: Configure OAuth & Permissions

Navigate to **OAuth & Permissions** in the left sidebar:

#### Bot Token Scopes (Required)

Add these scopes under **"Scopes" → "Bot Token Scopes"**:

```
app_mentions:read        # Read app mentions
channels:history           # View messages in channels
channels:join              # Join public channels
channels:read              # View channel information
chat:write                 # Send messages
chat:write.public          # Send messages to public channels
files:read                 # Read files
files:write                # Upload files
groups:history             # View messages in private channels
groups:read                # View private channel information
groups:write               # Manage private channels
im:history                 # View direct message history
im:read                    # View direct messages
im:write                   # Send direct messages
mpim:history               # View group DM history
mpim:read                  # View group DMs
mpim:write                 # Send group DMs
reactions:read             # View reactions
reactions:write            # Add/remove reactions
users:read                 # View user information
users:read.email           # View user email addresses
```

#### User Token Scopes (Optional - for advanced features)

```
channels:history
chat:write
files:read
files:write
users:read
```

### Step 3: Configure Event Subscriptions

Navigate to **Event Subscriptions**:

1. Enable **"Enable Events"** (toggle on)
2. For Socket Mode (recommended), skip the Request URL
3. Add these **Bot Events**:

```
app_mention                # When bot is mentioned
channel_created            # New channel created
channel_deleted            # Channel deleted
file_change                # File changes
file_created               # New file uploaded
file_deleted               # File deleted
file_shared                # File shared
file_unshared              # File unshared
group_deleted              # Private channel deleted
member_joined_channel      # User joins channel
member_left_channel        # User leaves channel
message.app_home           # Messages in App Home
message.channels           # Messages in channels
message.groups             # Messages in private channels
message.im                 # Direct messages
message.mpim               # Group direct messages
reaction_added             # Reaction added
reaction_removed           # Reaction removed
```

### Step 4: Enable Socket Mode

Navigate to **Socket Mode**:

1. Enable **"Socket Mode"** (toggle on)
2. Generate an **App-Level Token**:
   - Go to **Basic Information** → **App-Level Tokens**
   - Click **"Generate Token and Scopes"**
   - Name: `openclaw-socket`
   - Add scope: `connections:write`
   - Copy the token (starts with `xapp-`)

### Step 5: Install App to Workspace

Navigate to **Install App**:

1. Click **"Install to Workspace"**
2. Review permissions and click **"Allow"**
3. Copy the **Bot User OAuth Token** (starts with `xoxb-`)

### Step 6: Configure OpenClaw

Edit your `~/.openclaw/openclaw.json` file:

```json
{
  "channels": {
    "slack": {
      "enabled": true,
      "mode": "socket",
      "botToken": "xoxb-YOUR-BOT-TOKEN",
      "appToken": "xapp-YOUR-APP-TOKEN",
      "dmPolicy": "pairing",
      "allowFrom": ["*"],
      "groupPolicy": "open",
      "capabilities": {
        "interactiveReplies": true
      }
    }
  }
}
```

### Step 7: Invite Bot to Channels

In Slack:

```
/invite @YourBotName
```

Or via OpenClaw CLI:

```bash
openclaw message send --to #general --message "Hello from OpenClaw!"
```

### Security Configuration

#### DM Pairing (Recommended)

```json
{
  "channels": {
    "slack": {
      "dmPolicy": "pairing",
      "allowFrom": ["*"]
    }
  }
}
```

With `dmPolicy: pairing`, unknown senders receive a pairing code. Approve them with:

```bash
openclaw pairing approve slack <code>
```

#### Open DMs (Less Secure)

```json
{
  "channels": {
    "slack": {
      "dmPolicy": "open",
      "allowFrom": ["*"]
    }
  }
}
```

#### Restricted Access

```json
{
  "channels": {
    "slack": {
      "dmPolicy": "pairing",
      "allowFrom": ["@username1", "@username2", "U1234567890"]
    }
  }
}
```

---

## Configuration

### Full Config Example

```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "fireworks/accounts/fireworks/routers/kimi-k2p5-turbo"
      },
      "workspace": "/home/user/.openclaw/workspace"
    }
  },
  "gateway": {
    "mode": "local",
    "bind": "lan",
    "port": 18789,
    "auth": {
      "mode": "token",
      "token": "your-secure-token"
    }
  },
  "channels": {
    "slack": {
      "enabled": true,
      "mode": "socket",
      "botToken": "xoxb-xxx",
      "appToken": "xapp-xxx",
      "dmPolicy": "pairing",
      "allowFrom": ["*"]
    },
    "mattermost": {
      "enabled": true,
      "baseUrl": "https://mattermost.yourdomain.com",
      "botToken": "your-token",
      "dmPolicy": "pairing"
    }
  },
  "mcp": {
    "servers": {
      "slack": {
        "command": "npx",
        "args": ["-y", "@slack/mcp-server"],
        "env": {
          "SLACK_BOT_TOKEN": "xoxb-xxx",
          "SLACK_APP_TOKEN": "xapp-xxx"
        }
      }
    }
  }
}
```

### CLI Commands

```bash
# Status check
openclaw doctor

# Start gateway
openclaw gateway --port 18789 --verbose

# Send message
openclaw message send --to +1234567890 --message "Hello"

# Talk to agent
openclaw agent --message "Ship checklist" --thinking high

# List sessions
openclaw sessions list

# Compact context
openclaw compact

# Update
openclaw update
```

### Chat Commands (via Slack/Discord)

```
/status           - Show agent status
/new              - Start fresh conversation
/reset            - Reset session
/compact          - Compress context
/think <level>    - Set thinking level (low/medium/high)
/verbose on|off   - Toggle verbose mode
/trace on|off     - Toggle trace mode
/usage            - Show token usage
/restart          - Restart agent
/activation       - Set activation mode
```

---

## Troubleshooting

### Common Issues

#### Bot not responding to mentions

1. Verify bot is invited to channel: `/invite @botname`
2. Check `app_mentions:read` scope is added
3. Ensure `app_mention` event is subscribed
4. Regenerate tokens if changed

#### Socket Mode disconnects

```bash
# Check gateway logs
openclaw gateway --verbose

# Restart gateway
openclaw gateway restart
```

#### DM pairing not working

```bash
# Check current policy
openclaw config get channels.slack.dmPolicy

# List pending pairings
openclaw pairing list

# Approve manually
openclaw pairing approve slack <code>
```

#### Rate limiting

```json
{
  "channels": {
    "slack": {
      "rateLimit": {
        "messagesPerSecond": 1,
        "burstSize": 5
      }
    }
  }
}
```

### Verification Steps

```bash
# 1. Check config validity
openclaw doctor

# 2. Test Slack connection
openclaw gateway --verbose &
openclaw message send --to @yourusername --message "Test"

# 3. Verify channels
openclaw channels list

# 4. Check logs
tail -f ~/.openclaw/logs/gateway.log
```

### Getting Help

- **Docs:** https://docs.openclaw.ai
- **Discord:** https://discord.gg/clawd
- **GitHub Issues:** https://github.com/openclaw/openclaw/issues
- **ClawHub (Skills):** https://clawhub.ai

---

## Next Steps

1. [Install skills from ClawHub](https://clawhub.ai)
2. [Set up additional channels](https://docs.openclaw.ai/channels)
3. [Configure sandboxing](https://docs.openclaw.ai/gateway/sandboxing)
4. [Explore the Canvas feature](https://docs.openclaw.ai/platforms/mac/canvas)

---

*Last updated: 2026-04-22*
