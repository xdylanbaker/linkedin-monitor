# LinkedIn Monitor for OpenClaw

**LinkedIn inbox monitoring with zero duplicate notifications.**

Get notified when someone messages you on LinkedIn. Drafts replies in your voice. Never miss a lead.

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![OpenClaw](https://img.shields.io/badge/openclaw-skill-orange.svg)

---

## Features

- 🔔 **Hourly monitoring**: Checks your inbox every hour so you don't have to
- 🚫 **No duplicates**: Tracks what it's already told you about
- ✍️ **Draft replies**: Writes responses in your voice using your communication style file
- 🛡️ **Approval required**: Nothing gets sent without your say-so
- 🌐 **Browser-based**: Just uses your normal LinkedIn login
- ⚡ **Watchdog included**: Pings you if something breaks

### Coming Soon

- 🤝 **Connection request monitoring**: Auto-accept or decline based on your criteria
- 🔔 **Notification alerts**: Mentions, comments, profile views

---

## Quick Install

```bash
# 1. Install the skill
clawdhub install linkedin-monitor

# 2. Run setup
linkedin-monitor setup

# 3. Verify everything works
linkedin-monitor health

# 4. Enable hourly monitoring
linkedin-monitor enable
```

---

## Requirements

- [OpenClaw](https://github.com/openclaw/openclaw) installed and running
- A browser profile logged into LinkedIn
- A channel for alerts (Discord, Telegram, Slack, WhatsApp, or any OpenClaw channel)

---

## Setup Guide

### Step 1: Create a Browser Profile

LinkedIn Monitor uses OpenClaw's browser tool to check your inbox. You need a browser profile that's logged into LinkedIn.

**Option A: Use the default "clawd" profile**
```bash
# Start the clawd browser
openclaw browser start --profile clawd

# This opens a Chrome window. Log into LinkedIn manually.
# Keep this browser open — it needs to stay running.
```

**Option B: Create a custom profile**
```bash
# Create a profile named "linkedin"
openclaw browser start --profile linkedin

# Log into LinkedIn, then keep it open
```

> **Tip:** Put the browser on a second desktop/workspace so it doesn't interfere with your main browsing.

### Step 2: Run Setup

```bash
linkedin-monitor setup
```

This will ask you for:
- **Discord channel ID** — Where to send alerts (right-click channel → Copy ID)
- **Calendar link** — For booking meetings (e.g., cal.com/yourname)
- **Timezone** — Your timezone (e.g., America/New_York)

Configuration is saved to `~/.openclaw/linkedin-monitor/config.json`

### Step 3: Verify Health

```bash
linkedin-monitor health
```

You should see:
```
Dependencies:
✓ jq installed
✓ Browser profile ready

Authentication:
✓ LinkedIn logged in as: Your Name

Configuration:
✓ Config file exists
✓ Alert channel configured

Automation:
! Cron job not installed — run: linkedin-monitor enable
```

### Step 4: Test Manually

```bash
linkedin-monitor check
```

This runs one check cycle. If you have unread messages, it will report them.

### Step 5: Enable Hourly Monitoring

```bash
linkedin-monitor enable
```

This creates a cron job that runs every hour to check your inbox.

---

## How It Works

```
Every Hour:
┌─────────────────────────────────────────┐
│ 1. Check browser is running             │
│ 2. Open LinkedIn messaging tab          │
│ 3. Take snapshot of inbox               │
│ 4. Compare against seen messages        │
│ 5. If NEW message:                      │
│    → Draft reply in your voice          │
│    → Alert you on Discord               │
│    → Wait for approval                  │
│ 6. Update state file                    │
└─────────────────────────────────────────┘
```

**State is tracked in:** `~/.openclaw/linkedin-monitor/state/messages.json`

Each message gets a unique ID (`Name_Timestamp`). Once reported, it's added to `seenIds` and never reported again.

---

## Commands

| Command | Description |
|---------|-------------|
| `linkedin-monitor setup` | Interactive setup wizard |
| `linkedin-monitor health` | Check dependencies and auth |
| `linkedin-monitor check` | Run one monitoring cycle |
| `linkedin-monitor enable` | Enable hourly cron |
| `linkedin-monitor disable` | Disable cron |
| `linkedin-monitor status` | Show current state |
| `linkedin-monitor config` | View/edit configuration |
| `linkedin-monitor logs` | View activity logs |
| `linkedin-monitor reset` | Clear state (fresh start) |

---

## Approving Messages

When a new message arrives, you'll get an alert like:

```
📬 **John Smith**
> Hey, I saw your post about AI automation. Would love to chat!

**Draft reply:**
> Hey John, thanks for reaching out. Happy to connect — what specifically caught your interest?

Reply "send John" to approve.
```

**Commands:**
- `send John` — Send the draft as-is
- `send all` — Send all pending drafts
- `edit John Hey John, let's set up a call...` — Edit and send
- `skip John` — Discard the draft

---

## Configuration

Edit `~/.openclaw/linkedin-monitor/config.json`:

```json
{
  "autonomyLevel": 1,
  "alertChannel": "discord",
  "alertTarget": "YOUR_CHANNEL_ID",
  "calendarLink": "cal.com/yourname",
  "communicationStyleFile": "USER.md",
  "timezone": "America/New_York",
  "schedule": "0 * * * *",
  "browserProfile": "clawd"
}
```

### Supported Channels

| Channel | alertChannel | alertTarget |
|---------|--------------|-------------|
| Discord | `discord` | Channel ID |
| Telegram | `telegram` | Chat ID |
| Slack | `slack` | Channel ID or name |
| WhatsApp | `whatsapp` | Chat ID or phone |
| Signal | `signal` | Phone number |

### Autonomy Levels

**Level 0: Monitor Only**
Just watches your inbox and alerts you when something new comes in. No drafts, no replies. Good if you just want notifications.

**Level 1: Draft + Approve** (default)
Drafts a reply for each new message, but waits for you to approve before sending anything. You stay in full control.

**Level 2: Auto-reply Simple**
Handles the easy stuff automatically:
- "Thanks for connecting" messages
- Simple scheduling requests (sends your calendar link)
- Generic greetings

Still alerts you for anything that needs a real response.

**Level 3: Full Autonomous**
Replies as you, handles scheduling, books meetings on your calendar. Only pings you for edge cases or when it's unsure. This is the "set it and forget it" mode.

Change your level with:
```bash
linkedin-monitor config autonomyLevel 2
```

---

## Communication Style

Drafts are generated using your communication style from `USER.md` in your OpenClaw workspace.

If you don't have a `USER.md`, create one with your preferences:

```markdown
# USER.md

## Communication Style
- Keep messages short (3-4 sentences)
- Be friendly but professional
- No corporate jargon
- Always offer value first
```

### Pro tip: Use your LinkedIn data export

Want drafts that really sound like you? Request your LinkedIn data export (Settings > Data privacy > Get a copy of your data). This gives you your full message history and post history.

Feed that to an LLM and ask it to analyze your communication patterns, tone, vocabulary, and style. Then add those insights to your `USER.md`. The more detailed your style guide, the better the drafts.

---

## Troubleshooting

### "Browser not running"
```bash
openclaw browser start --profile clawd
# Then log into LinkedIn
```

### "LinkedIn logged out"
Open the clawd browser and log back into LinkedIn manually. The browser is usually on your second desktop.

### "Duplicate notifications"
```bash
linkedin-monitor reset
# This clears state — next check will see all messages as "new"
```

### "Cron not running"
```bash
linkedin-monitor disable
linkedin-monitor enable
# Check: crontab -l
```

### "Watchdog alert: Monitor may be down"
```bash
linkedin-monitor health
# Fix whatever's broken, then:
linkedin-monitor check
```

---

## Files

```
~/.openclaw/linkedin-monitor/
├── config.json          # Your settings
├── state/
│   └── messages.json    # Seen message IDs
└── logs/
    └── activity.log     # Activity history
```

---

## Uninstall

```bash
# Disable cron
linkedin-monitor disable

# Remove the skill
clawdhub uninstall linkedin-monitor

# (Optional) Remove data
rm -rf ~/.openclaw/linkedin-monitor
```

---

## Contributing

Found a bug? Have a feature request? Open an issue on GitHub.

---

## License

MIT License — use it however you want.

---

## Credits

Built by [Dylan Baker](https://github.com/dylanbaker) / [lilAgents](https://lilagents.com)

Part of the [OpenClaw](https://clawd.bot) ecosystem.
