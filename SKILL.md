---
name: new-agent
description: Create a new OpenClaw agent and connect it to a messaging channel (Telegram, Discord, Slack, Feishu, WhatsApp, Signal, Google Chat). Includes workspace scaffolding, channel configuration, and gateway binding.
---

# New Agent

Add a new agent to your OpenClaw gateway with a dedicated workspace and messaging channel.

## When to Use

- User wants to add a new AI agent or bot
- User wants to connect a bot to a messaging platform
- User provides an agent name and channel credentials

## Overview

Adding a new agent involves four parts:
1. **Workspace** — Create identity, personality, and memory files
2. **Registration** — Register the agent with the CLI
3. **Channel** — Add account credentials and routing binding to `openclaw.json`
4. **Verify** — Restart gateway, verify, and pair

## Required Information

| Field | Example |
|-------|---------|
| Agent name | "Luna" |
| Channel | telegram / discord / slack / feishu / whatsapp / signal / googlechat |
| Credentials | Bot token, app secret, or QR scan |

## Step 1: Workspace

Run the helper script:

```bash
./scripts/setup-agent.sh {name}
```

This creates a workspace at `~/.openclaw/workspace-groups/{name}/` with:
- `IDENTITY.md` — Name, role, emoji
- `SOUL.md` — Personality and behavior
- `AGENTS.md` — Startup instructions
- `USER.md` — Owner info

Or create the directory and files manually.

## Step 2: Registration

Register the agent with the CLI:

```bash
openclaw agents add {name}-agent \
  --workspace ~/.openclaw/workspace-groups/{name} \
  --non-interactive
```

> **Important:** Always pass `--non-interactive` and `--workspace` for automation. Without these, the CLI opens an interactive prompt.

This adds the agent to `agents.list` in `openclaw.json`.

## Step 3: Channel Configuration

Each channel needs **two things** in `openclaw.json`:
1. An **account entry** under `channels.{channel}.accounts`
2. A **binding** in the **top-level `bindings` array**

> ⚠️ The `bindings` array is at the **root level** of `openclaw.json`, NOT under `agents`.

### 3a. Account Entry

Add under `channels.{channel}.accounts.{name}`:

**Telegram:**
```json
{
  "dmPolicy": "pairing",
  "botToken": "YOUR_BOT_TOKEN",
  "groupPolicy": "open",
  "streaming": "partial"
}
```

**Discord:**
```json
{
  "token": "YOUR_BOT_TOKEN"
}
```

**Slack:**
```json
{
  "mode": "socket",
  "appToken": "xapp-...",
  "botToken": "xoxb-..."
}
```

**Feishu / Lark:**
```json
{
  "appId": "YOUR_APP_ID",
  "appSecret": "YOUR_APP_SECRET"
}
```
For Lark (global), add `"domain": "lark"`.

**WhatsApp / Signal** — No account entry needed; use interactive login:
```bash
openclaw channels login --channel whatsapp --account {name}
openclaw channels login --channel signal --account {name}
```

### 3b. Binding (Top-Level)

Add to the root `bindings` array:

```json
{
  "agentId": "{name}-agent",
  "match": {
    "channel": "{channel}",
    "accountId": "{name}"
  }
}
```

### 3c. Agent-to-Agent (Optional)

To allow other agents to communicate with the new agent, add `"{name}-agent"` to `tools.agentToAgent.allow`.

## Step 4: Verify & Pair

```bash
# Restart to apply config
openclaw gateway restart

# Check agent and bindings
openclaw agents list --bindings

# Probe channel health
openclaw channels status --probe
```

For DM-based channels (Telegram, Discord, etc.), the owner sends `/start` to the bot, then approves pairing:

```bash
openclaw pairing approve {channel} {CODE}
```

## Notes

- All agents share existing model credentials — no extra API keys needed
- One channel is enough to bring an agent online
- Add more channels later by repeating Step 3
- The default model comes from `agents.defaults.model.primary` in your config
