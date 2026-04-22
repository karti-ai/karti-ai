# Hermes Agent Installation Guide

> **Repository:** https://github.com/nousresearch/hermes-agent  
> **Description:** The self-improving AI agent built by Nous Research  
> **Stars:** 111k+ | **Language:** Python (87.4%) / TypeScript (8.6%)  
> **License:** MIT

---

## Table of Contents

1. [What is Hermes Agent?](#what-is-hermes-agent)
2. [Quick Install](#quick-install)
3. [Slack MCP Full Setup](#slack-mcp-full-setup)
4. [Configuration](#configuration)
5. [Gateway & Messaging](#gateway--messaging)
6. [Troubleshooting](#troubleshooting)

---

## What is Hermes Agent?

Hermes Agent is the **only agent with a built-in learning loop** — it creates skills from experience, improves them during use, nudges itself to persist knowledge, and builds a deepening model of who you are across sessions.

### Key Features

- **Self-Improving** - Creates skills from experience, auto-improves them
- **Learning Loop** - Periodic nudges to persist knowledge
- **Memory Search** - FTS5 session search with LLM summarization
- **Multi-Platform** - Telegram, Discord, Slack, WhatsApp, Signal
- **Scheduled Automations** - Built-in cron scheduler
- **Parallel Subagents** - Spawn isolated workers for parallel tasks
- **Provider Agnostic** - Nous Portal, OpenRouter, NVIDIA NIM, OpenAI, local models
- **6 Terminal Backends** - Local, Docker, SSH, Daytona, Singularity, Modal

---

## Quick Install

### One-Line Install (Recommended)

```bash
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
```

Works on: Linux, macOS, WSL2, Android (Termux)

### Post-Install Setup

```bash
# Reload shell
source ~/.bashrc    # or: source ~/.zshrc

# Start Hermes
hermes
```

### Manual Install (Development)

```bash
# Clone repository
git clone https://github.com/NousResearch/hermes-agent.git
cd hermes-agent

# Run setup script (installs uv, creates venv, installs dependencies)
./setup-hermes.sh

# Or manual path:
curl -LsSf https://astral.sh/uv/install.sh | sh
uv venv venv --python 3.11
source venv/bin/activate
uv pip install -e ".[all,dev]"

# Create symlink
ln -s $(pwd)/hermes ~/.local/bin/hermes

# Run
./hermes
```

### Termux (Android)

See [Termux Guide](https://hermes-agent.nousresearch.com/docs/getting-started/termux) for Android-specific setup.

```bash
# Note: Full install currently has Android-incompatible voice dependencies
# Use curated .[termux] extra instead of .[all]
```

---

## Slack MCP Full Setup

### Step 1: Create Slack App

1. Visit [api.slack.com/apps](https://api.slack.com/apps)
2. Click **"Create New App"**
3. Choose **"From scratch"**
4. App Name: `Hermes Agent`
5. Select your workspace
6. Click **"Create App"**

### Step 2: Configure OAuth Scopes

Navigate to **OAuth & Permissions**:

#### Bot Token Scopes Required

```
app_mentions:read         # Read bot mentions
channels:history          # View public channel messages
channels:join             # Join public channels
channels:read             # View channel info
chat:write                # Send messages
chat:write.public         # Send to public channels
files:read                # Read shared files
files:write               # Upload files
groups:history            # View private channel messages
groups:read               # View private channel info
groups:write              # Manage private channels
im:history                # View DM history
im:read                   # View DMs
im:write                  # Send DMs
mpim:history              # View group DM history
mpim:read                 # View group DMs
mpim:write                # Send group DMs
reactions:read            # View emoji reactions
reactions:write           # Add/remove reactions
users:read                # View workspace members
users:read.email          # View member emails
```

### Step 3: Enable Event Subscriptions

Go to **Event Subscriptions**:

1. Toggle **"Enable Events"** ON
2. For Socket Mode, skip Request URL
3. Add these **Bot Events**:

```
app_mention               # Bot is mentioned
channel_created           # New channel created
channel_deleted           # Channel deleted
file_change               # File modified
file_created              # File uploaded
file_deleted              # File deleted
file_shared               # File shared
file_unshared             # File unshared
group_deleted             # Private channel deleted
member_joined_channel     # User joins channel
member_left_channel       # User leaves channel
message.app_home          # App Home messages
message.channels          # Public channel messages
message.groups            # Private channel messages
message.im                # Direct messages
message.mpim              # Group DMs
reaction_added            # Reaction added
reaction_removed          # Reaction removed
```

### Step 4: Enable Socket Mode

Navigate to **Socket Mode**:

1. Toggle **"Socket Mode"** ON
2. Go to **Basic Information** → **App-Level Tokens**
3. Click **"Generate Token and Scopes"**
4. Token Name: `hermes-socket`
5. Scope: `connections:write`
6. Click **"Generate"**
7. Copy the `xapp-` token

### Step 5: Install to Workspace

1. Go to **OAuth & Permissions**
2. Click **"Install to Workspace"**
3. Review and click **"Allow"**
4. Copy the **Bot User OAuth Token** (`xoxb-`)

### Step 6: Configure Hermes

Create/edit `~/.hermes/config.yaml`:

```yaml
# ~/.hermes/config.yaml

# Model configuration
model:
  provider: anthropic  # or: openai, openrouter, nous, etc.
  name: claude-3-5-sonnet-20241022
  api_key: sk-ant-api03-xxx

# Gateway configuration
gateway:
  enabled: true
  platforms:
    slack:
      enabled: true
      bot_token: xoxb-YOUR-BOT-TOKEN
      app_token: xapp-YOUR-APP-TOKEN
      socket_mode: true
      # Security
      dm_policy: pairing  # or: open, restricted
      allowed_users:
        - "@username"
        - "U12345678"
      
# MCP servers
mcp:
  servers:
    slack:
      command: npx
      args: ["-y", "@modelcontextprotocol/server-slack"]
      env:
        SLACK_BOT_TOKEN: xoxb-YOUR-BOT-TOKEN
        SLACK_APP_TOKEN: xapp-YOUR-APP-TOKEN
```

### Step 7: Environment Variables

```bash
# Add to ~/.bashrc or ~/.zshrc
export SLACK_BOT_TOKEN="xoxb-xxx"
export SLACK_APP_TOKEN="xapp-xxx"

# Hermes-specific
export HERMES_CONFIG_PATH="~/.hermes/config.yaml"
export HERMES_MODEL_PROVIDER="anthropic"
export HERMES_MODEL_API_KEY="sk-ant-xxx"
```

### Step 8: Invite Bot & Test

```bash
# In Slack channel:
/invite @Hermes

# Or via CLI
hermes message send --channel #general --message "Hello from Hermes!"
```

---

## Configuration

### Full Config Example (`~/.hermes/config.yaml`)

```yaml
# Core settings
model:
  provider: anthropic
  name: claude-3-5-sonnet-20241022
  api_key: sk-ant-api03-xxx
  temperature: 0.7
  max_tokens: 4096

# Alternative providers
models:
  fast:
    provider: openai
    name: gpt-4o-mini
    api_key: sk-xxx
  local:
    provider: ollama
    name: llama3.2
    base_url: http://localhost:11434

# Gateway / Messaging
gateway:
  enabled: true
  host: 0.0.0.0
  port: 8080
  
  platforms:
    slack:
      enabled: true
      bot_token: xoxb-xxx
      app_token: xapp-xxx
      socket_mode: true
      dm_policy: pairing
      allowed_channels: ["*"]
      
    telegram:
      enabled: true
      bot_token: xxx:yyy
      
    discord:
      enabled: true
      bot_token: xxx

# MCP integration
mcp:
  enabled: true
  servers:
    slack:
      command: npx
      args: ["-y", "@modelcontextprotocol/server-slack"]
      env:
        SLACK_BOT_TOKEN: xoxb-xxx
        SLACK_APP_TOKEN: xapp-xxx
        
    github:
      command: npx
      args: ["-y", "@modelcontextprotocol/server-github"]
      env:
        GITHUB_PERSONAL_ACCESS_TOKEN: ghp-xxx

# Skills
skills:
  auto_create: true
  path: ~/.hermes/skills
  
# Memory
memory:
  enabled: true
  path: ~/.hermes/memory
  search_enabled: true
  
# Cron jobs
cron:
  enabled: true
  jobs:
    - name: daily-report
      schedule: "0 9 * * *"
      command: "generate daily report"
      platform: slack
      channel: "#reports"

# Logging
logging:
  level: info
  path: ~/.hermes/logs
```

### CLI Commands

```bash
# Start interactive CLI
hermes

# Start gateway (for messaging platforms)
hermes gateway
hermes gateway start
hermes gateway setup

# Configuration
hermes config set model.provider anthropic
hermes config get model.provider
hermes config edit

# Model management
hermes model                    # Interactive model selection
hermes model list               # List available models
hermes model set anthropic:claude-3-5-sonnet

# Tools
hermes tools                    # Configure enabled tools
hermes tools list
hermes tools enable slack

# Update & Maintenance
hermes update                   # Update to latest version
hermes doctor                   # Diagnose issues
hermes version

# Migration (from OpenClaw)
hermes claw migrate             # Full migration
hermes claw migrate --dry-run   # Preview changes
hermes claw migrate --preset user-data
```

### CLI vs Messaging Interface

| Action | CLI | Messaging |
|--------|-----|-------------|
| Start chatting | `hermes` | Run `hermes gateway`, then message bot |
| New conversation | `/new` or `/reset` | `/new` or `/reset` |
| Change model | `/model [provider:model]` | `/model [provider:model]` |
| Set personality | `/personality [name]` | `/personality [name]` |
| Retry/Undo | `/retry`, `/undo` | `/retry`, `/undo` |
| Compress context | `/compress` | `/compress` |
| Check usage | `/usage`, `/insights [--days N]` | `/usage`, `/insights [days]` |
| Browse skills | `/skills` or `/<skill-name>` | `/skills` or `/<skill-name>` |
| Stop current work | `Ctrl+C` | `/stop` |
| Platform status | `/platforms` | `/status`, `/sethome` |

---

## Gateway & Messaging

### Starting the Gateway

```bash
# Start gateway daemon
hermes gateway start

# With verbose logging
hermes gateway --verbose

# Setup wizard
hermes gateway setup
```

### Platform-Specific Setup

#### Slack Security Modes

**Pairing Mode (Recommended):**
```yaml
gateway:
  platforms:
    slack:
      dm_policy: pairing
      allowed_users: ["*"]  # Any user after pairing
```

Unknown users get a pairing code. Approve with:
```bash
hermes pairing approve slack <code>
```

**Open Mode:**
```yaml
gateway:
  platforms:
    slack:
      dm_policy: open
      allowed_users: ["*"]
```

**Restricted Mode:**
```yaml
gateway:
  platforms:
    slack:
      dm_policy: restricted
      allowed_users:
        - "@alice"
        - "@bob"
        - "U12345678"
```

### Cron Scheduling

```yaml
cron:
  enabled: true
  jobs:
    # Daily standup report
    - name: standup
      schedule: "0 9 * * 1-5"
      command: "generate standup report from git commits"
      platform: slack
      channel: "#standup"
      
    # Weekly summary
    - name: weekly
      schedule: "0 17 * * 5"
      command: "summarize week's work"
      platform: slack
      channel: "#general"
      
    # System monitoring
    - name: health-check
      schedule: "*/30 * * * *"
      command: "check system health and alert if issues"
      platform: slack
      channel: "#alerts"
```

---

## Troubleshooting

### Installation Issues

```bash
# Verify Python version (3.11+ required)
python --version

# Check uv installation
which uv
uv --version

# Reinstall with verbose output
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash -x
```

### Gateway Won't Start

```bash
# Check if port is in use
lsof -i :8080

# Check config validity
hermes doctor

# View logs
tail -f ~/.hermes/logs/gateway.log

# Restart with debug logging
hermes gateway --verbose --debug
```

### Slack Connection Issues

```bash
# Verify tokens
echo $SLACK_BOT_TOKEN
echo $SLACK_APP_TOKEN

# Test connection
hermes gateway --test-connection slack

# Check allowed channels
hermes config get gateway.platforms.slack.allowed_channels
```

### MCP Server Errors

```bash
# Test MCP server manually
npx -y @modelcontextprotocol/server-slack

# Check MCP config
hermes config get mcp

# Restart MCP
hermes mcp restart
```

### Migration from OpenClaw

```bash
# Preview migration
hermes claw migrate --dry-run

# Full migration
hermes claw migrate

# Migrate specific data only
hermes claw migrate --preset user-data    # No secrets
hermes claw migrate --preset memories     # Only memories
hermes claw migrate --preset skills       # Only skills

# Force overwrite
hermes claw migrate --overwrite
```

### Common Errors

| Error | Solution |
|-------|----------|
| `ModuleNotFoundError` | Reinstall: `uv pip install -e ".[all]"` |
| `Permission denied` | Check `~/.hermes` permissions: `chmod 755 ~/.hermes` |
| `Connection refused` | Check gateway is running: `hermes gateway status` |
| `Invalid token` | Regenerate tokens at api.slack.com |
| `missing_scope` | Add missing scope in OAuth settings |
| `rate_limited` | Add rate limiting to config |

### Debug Mode

```bash
# Enable debug logging
export HERMES_DEBUG=1
hermes

# Or verbose flag
hermes --verbose
hermes gateway --verbose
```

### Reset Everything

```bash
# Backup first
cp -r ~/.hermes ~/.hermes.backup

# Reset config
rm -rf ~/.hermes/config.yaml
hermes setup

# Full reset (keeps memories)
hermes reset

# Nuclear option (everything)
rm -rf ~/.hermes
hermes setup
```

---

## Getting Help

- **Documentation:** https://hermes-agent.nousresearch.com/docs/
- **Discord:** https://discord.gg/NousResearch
- **GitHub Issues:** https://github.com/NousResearch/hermes-agent/issues
- **GitHub Discussions:** https://github.com/NousResearch/hermes-agent/discussions
- **Skills Hub:** https://agentskills.io

---

## Advanced Configuration

### RL Training (Optional)

```bash
# For Tinker-Atropos RL integration
git submodule update --init tinker-atropos
uv pip install -e "./tinker-atropos"
```

### Custom Terminal Backend

```yaml
# ~/.hermes/config.yaml
terminal:
  backend: docker  # or: local, ssh, daytona, singularity, modal
  docker:
    image: hermes-sandbox:latest
    volumes:
      - /home/user/projects:/workspace
```

### Batch Trajectory Generation

```bash
# For research/training data generation
python batch_runner.py --config datagen-config.yaml
```

---

*Last updated: 2026-04-22*
