---
title: "Meet OpenClaw: Running a Personal AI Assistant on a Mac Mini"
date: 2026-02-28
draft: false
tags: ["openclaw", "self-hosted", "mac-mini", "ai"]
categories: ["Infrastructure"]
summary: "How I set up OpenClaw — an open-source AI assistant — on a Mac mini M4 Pro, and why running your own AI beats relying on the cloud."
ShowToc: true
TocOpen: true
---

## What is OpenClaw?

[OpenClaw](https://github.com/openclaw/openclaw) is an open-source personal AI assistant framework. Think of it as a bridge between LLM providers (Anthropic, OpenAI, etc.) and your daily life — messaging platforms, calendars, files, smart home devices, and more.

Unlike cloud-hosted assistants that live behind someone else's API, OpenClaw runs on **your** hardware. You own the data, control the integrations, and decide what it can access.

It's built on Node.js, speaks to multiple messaging channels (Telegram, Discord, Signal, WhatsApp, Slack), and can be extended with **skills** — modular packages that teach it new tricks like managing GitHub issues, controlling smart home devices, or searching the web.

## The Hardware: Mac Mini M4 Pro

Here's what's running the show:

| Component | Spec |
|-----------|------|
| **Machine** | Mac mini (2024) |
| **Chip** | Apple M4 Pro |
| **Memory** | 64 GB unified |
| **Storage** | 1 TB SSD |
| **OS** | macOS Tahoe 26.3 |
| **Uptime** | 24/7, headless |

### Why a Mac Mini?

A few reasons:

1. **Power efficiency** — the M4 Pro sips power compared to a traditional server. It sits on my desk drawing maybe 10-15W idle. My electricity bill doesn't care.
2. **ARM performance** — Apple Silicon is genuinely fast for this workload. Node.js runs natively on ARM64, and the unified memory architecture means no bottlenecks when juggling multiple AI sessions.
3. **macOS ecosystem** — access to Apple Notes, Reminders, Calendar via native CLIs. If you're in the Apple ecosystem already, this is a natural fit.
4. **Quiet** — it's literally silent. No fan noise, no spinning disks. It sits next to me and I forget it's there.

Could I run this on a Raspberry Pi? Probably — OpenClaw is lightweight. But the Mac mini gives me headroom for running local models, multiple parallel sessions, and whatever else I throw at it.

## How OpenClaw Works

The architecture is straightforward:

```
[Telegram / Discord / Signal]
        ↓
    OpenClaw Gateway
        ↓
   Session Manager
        ↓
  LLM Provider (Anthropic, OpenAI, etc.)
        ↓
  Skills & Tools (web search, calendar, GitHub, etc.)
```

The **Gateway** is the always-on daemon. It:
- Listens to incoming messages from configured channels
- Manages sessions (one per chat/user)
- Routes messages to the configured LLM
- Exposes tools the LLM can call (file access, web search, shell commands, etc.)
- Runs cron jobs and heartbeat checks

### Sessions

Each conversation gets its own **session** with isolated context. A direct Telegram chat is one session, a group chat is another, a cron job spawns its own isolated session. They don't leak into each other.

### Skills

Skills are the real power of OpenClaw. They're packaged instructions + scripts that teach the assistant how to use specific tools. Some examples from my setup:

- **`gog`** — Google Workspace CLI for Gmail, Calendar, Drive
- **`github`** — full GitHub operations via `gh` CLI
- **`weather`** — forecasts via wttr.in
- **`coding-agent`** — spawns Codex or Claude Code for complex coding tasks
- **`apple-reminders`** — manages Apple Reminders via a native CLI

You drop a skill into the skills directory, and the assistant picks it up. No retraining, no fine-tuning — it reads the skill's `SKILL.md` and follows the instructions.

## My Configuration

Here's a simplified view of how I have things wired up:

### Messaging
- **Telegram** as the primary interface — it's fast, supports rich formatting, and works everywhere
- Direct messages for personal tasks
- Group chats where the bot participates (but knows when to stay quiet)

### Models
- **Anthropic Claude** as the primary model (Sonnet for daily tasks, Opus for complex work)
- **MiniMax M2.5** as the default model for cost-efficiency
- Model switching on the fly via `/model` command

### Cron & Automation
- Periodic heartbeat checks (email, calendar, weather)
- Scheduled tasks (weekly family activity suggestions, reminders)
- Background sub-agents for long-running work

### Memory
The assistant maintains its own memory through markdown files:
- **Daily notes** (`memory/YYYY-MM-DD.md`) — raw logs of what happened
- **Long-term memory** (`MEMORY.md`) — curated, distilled knowledge
- **Identity files** (`SOUL.md`, `USER.md`) — who the assistant is and who it's helping

This is surprisingly effective. The assistant wakes up each session, reads its memory files, and picks up where it left off. It's not perfect — it's more like a journal than a brain — but it works.

## What I Use It For

Day to day:

- **Quick lookups** — weather, web searches, "what's on my calendar today?"
- **Email triage** — "any urgent emails?" without opening Gmail
- **Reminders and scheduling** — "remind me to call the dentist tomorrow at 10"
- **Coding help** — spawns sub-agents for PR reviews, bug fixes, new features
- **Family logistics** — weekend activity ideas, printing craft sheets for the kids
- **Home automation** — (work in progress)

The killer feature isn't any single capability — it's that everything is in **one place**. I message Telegram, and the assistant figures out which tool to use.

## Getting Started

If you want to try OpenClaw yourself:

```bash
# Install
npm install -g openclaw

# Initialize
openclaw init

# Configure (edit ~/.openclaw/config.yaml)
# Add your Telegram bot token, LLM API keys, etc.

# Start
openclaw gateway start
```

The [documentation](https://docs.openclaw.ai) covers the details. The [Discord community](https://discord.com/invite/clawd) is active and helpful.

### Minimum Requirements

- **Node.js 18+**
- **Any machine that stays on** — Mac, Linux, Raspberry Pi, VPS
- **An LLM API key** (Anthropic, OpenAI, or others)
- **A messaging platform** (Telegram is the easiest to start with)

## What's Next

In upcoming posts, I'll dig into:
- Setting up Telegram integration (and common pitfalls)
- Writing custom skills
- Running local LLMs alongside cloud models
- Memory architecture and how the assistant "remembers"

For now — if you're tired of cloud-only assistants and want something that runs on your own hardware, give OpenClaw a look. It's open source, actively developed, and genuinely useful.

---

*This blog itself is hosted on Cloudflare Pages and built with [Hugo](https://gohugo.io/) + [PaperMod](https://github.com/adityatelange/hugo-PaperMod). Because static sites are still the best sites.*
