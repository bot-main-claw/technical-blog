---
title: "Setting Up OpenClaw with Telegram: Configuration & Troubleshooting"
date: 2025-10-23
draft: false
tags: ["openclaw", "telegram", "self-hosted", "tutorial"]
categories: ["Guides"]
summary: "A practical guide to connecting OpenClaw to Telegram — from bot creation to group chat permissions, with solutions to every common problem I hit along the way."
ShowToc: true
TocOpen: true
---

OpenClaw is a powerful AI assistant framework that runs on your own hardware. In this guide, I'll walk through setting it up with Telegram as the primary messaging interface, and share solutions to the common problems I ran into during setup.

## Why Telegram?

Out of all the messaging platforms OpenClaw supports (Discord, Signal, WhatsApp, Slack, IRC…), Telegram is the easiest to get started with:

- **Bot API is straightforward** — create a bot in minutes via @BotFather
- **Rich formatting** — markdown, inline buttons, reactions, voice messages
- **Works everywhere** — mobile, desktop, web
- **No approval process** — unlike WhatsApp Business API, you don't need to apply for anything

It's what I use as my daily driver for talking to OpenClaw.

## Prerequisites

Before you start, you'll need:

- **Node.js 18+** installed on your machine
- **A Telegram bot token** (we'll create one below)
- **Your Telegram user ID** (for the allowlist)
- **OpenClaw installed** — see the [introduction post](/posts/openclaw-introduction/) for setup basics

## Step 1: Create Your Telegram Bot

Open Telegram and message [@BotFather](https://t.me/BotFather):

1. Send `/newbot`
2. Choose a display name (e.g., "My OpenClaw")
3. Choose a username ending in `bot` (e.g., `my_openclaw_bot`)
4. BotFather gives you a token like `7123456789:AAF...` — save this

A few settings to configure right away via BotFather:

```
/setprivacy → Disable  (so the bot can read all group messages)
/setcommands → Set your commands (optional)
```

## Step 2: Find Your Telegram User ID

You need your numeric user ID for the allowlist. The easiest way:

1. Message [@userinfobot](https://t.me/userinfobot) on Telegram
2. It replies with your user ID (a number like `5706366865`)

## Step 3: Configure OpenClaw

Edit your OpenClaw config (`~/.openclaw/config.yaml`) and add the Telegram section:

```yaml
telegram:
  enabled: true
  bot_token: "7123456789:AAFxxxxxxxxxxxxxxxxxxxxxxxxxx"
  allowed_users:
    - 5706366865  # Your Telegram user ID
```

Then start (or restart) the gateway:

```bash
openclaw gateway start
```

Send a message to your bot in Telegram — it should respond. If it does, you're done with the basics. 🎉

## Common Problems & Solutions

Here's every issue I hit and how I fixed them.

### 🔴 Bot doesn't see messages in group chats

**What happens:** You add the bot to a group, send a message, and… nothing. The bot only responds when you @mention it.

**Why:** By default, Telegram bots in groups only receive messages that mention them or start with a `/command`. This is the "privacy mode" setting.

**Fix — two options:**

1. **Always @mention the bot** — works but gets tedious
2. **Disable privacy mode** (recommended):
   - Message @BotFather: `/setprivacy`
   - Select your bot
   - Choose `Disable`
   - **Remove and re-add** the bot to existing groups (privacy mode changes don't apply retroactively!)

### 🔴 "Chat not found" errors

**What happens:** OpenClaw logs show errors about not finding the chat.

**Why:** Usually a config issue — wrong token or the user isn't allowlisted.

**Fix:**

- Double-check the bot token in your config (no extra spaces, no missing characters)
- Verify your user ID is in `allowed_users`
- Start a **private chat** with your bot first — send `/start`. Telegram requires this initial interaction before the bot can message you

### 🔴 Bot responds only to /commands

**What happens:** In a group, the bot reacts to `/help` but ignores regular messages.

**Why:** The bot doesn't have admin rights or privacy mode is still enabled.

**Fix:**

1. Go to **Group Settings → Administrators**
2. Add your bot as admin
3. Enable **"Can read all group messages"** (not just "Can reply")
4. If privacy mode was recently changed, remove and re-add the bot

### 🔴 Connection timeouts

**What happens:** OpenClaw starts but can't reach the Telegram API. Logs show timeout errors.

**Why:** Network issue between your machine and Telegram's servers.

**Fix:**

- Check your internet connection (obvious but worth verifying)
- Ensure **port 443 (HTTPS)** is open — some corporate networks or firewalls block it
- If you're behind a restrictive firewall, try a different network or use a VPN
- Some hosting providers (certain VPS regions) have Telegram API blocked — consider moving to a different host

### 🔴 Bot works in DM but not in groups

**What happens:** Private messages work fine, but the bot is silent in groups.

**Why:** Usually a combination of privacy mode and missing admin permissions.

**Fix:** Apply both fixes above:
1. Disable privacy mode via @BotFather
2. Make the bot a group admin with message reading permissions
3. Remove and re-add the bot to the group

## Testing Your Setup

Once everything is configured, verify these scenarios in order:

1. **DM the bot** → should respond immediately
2. **Add bot to a group and @mention it** → should respond
3. **Make bot admin, send a message without @** → should respond to all messages
4. **Send a voice message** → should transcribe and respond (if you have a transcription skill configured)

## Tips for Daily Use

A few things I've learned from using Telegram as my primary OpenClaw interface:

- **Pin the bot chat** — you'll message it a lot, keep it at the top
- **Use reply-to** — when the bot sends multiple messages, reply to a specific one for context
- **Group chats are great for family/team** — the bot can participate naturally without taking over
- **Telegram Desktop** for long conversations — the mobile app is fine, but desktop is better for code blocks and longer outputs

## What's Next

With Telegram connected, you've got the foundation. From here you can:

- Add skills (web search, calendar, email, GitHub)
- Set up cron jobs for periodic checks
- Configure multiple channels (add Discord or Signal alongside Telegram)
- Set up group chats where the bot participates

The bot gets more useful the more you connect to it. Start with the basics and expand from there.

---

*Having trouble with something not covered here? Check the [OpenClaw docs](https://docs.openclaw.ai) or join the [Discord community](https://discord.com/invite/clawd).*
