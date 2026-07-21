# Morning Brief

Wake up to a brief built for you: today's focus, open loops from yesterday, and one writing prompt to start your thinking — before the feed starts thinking for you.

Three levels. Level 1 works the minute you install it. Levels 2 and 3 are optional upgrades.

---

## When to use

- Every morning, ideally before opening any other app
- Say "morning brief" or "run my brief" in Claude Code
- Works best paired with the Life Wiki skill (the brief reads it), but runs fine without it

## Prerequisites

**Level 1 (core): none.** Claude Code and this file. No API keys, no subscriptions.

**Level 2 (scheduled):** none extra — uses Claude Code's built-in scheduled tasks.

**Level 3 (phone notification via Telegram):** a free Telegram account and a bot token (setup below, 10 minutes, free).

## One-time setup

1. Create a file called `context/priorities.md` in your workspace with three sections:

```markdown
# Priorities

## This quarter
(2-3 lines: what you're building toward)

## This week
(the 1-3 things that matter this week)

## Daily non-negotiables
(e.g. 30 min writing, training, no phone before 9)
```

2. That's it. The brief reads this file every morning. Update it whenever your priorities change — the brief stays current because the file is.

## How it works (Level 1)

When you ask for your brief, Claude:

1. Reads `context/priorities.md`
2. Reads yesterday's brief from `outputs/briefs/` (if one exists) and carries forward anything marked unfinished
3. Reads your wiki's recent log entries (if you have the Life Wiki installed) for context on what you've been thinking about
4. Writes today's brief to `outputs/briefs/YYYY-MM-DD.md` and shows it to you

**The brief format (keep it under 20 lines — it's a brief, not a report):**

```markdown
# Tuesday, 21 July

**Focus today:** (one line — the single thing that matters most, pulled from this week's priorities)

**Carried over:** (anything unfinished from yesterday's brief; omit if none)

**Writing prompt:** (one question to journal on, connected to what you're
building or thinking about — never generic. "What would make this week's
launch feel finished?" not "What are you grateful for?")

**One reminder:** (your daily non-negotiables, one line)
```

Rules for Claude when writing the brief:
- Everything traces to the user's files. Never invent tasks, events, or commitments.
- The writing prompt is the heart of it — make it specific to what they're actually working on right now.
- No motivational-poster language. Plain words, like a sharp friend texting you.
- If `priorities.md` is missing, ask the user to fill it in (show the template) — don't generate a generic brief.

## Level 2 — schedule it

Ask Claude Code to schedule the brief as a recurring task ("run my morning brief every day at 6:30am"). It generates while you sleep and is waiting in `outputs/briefs/` when you sit down.

## Level 3 — get it on your phone (Telegram)

This is the version where your phone lights up with the brief before you wake up.

1. In Telegram, message **@BotFather** → `/newbot` → follow the prompts → copy the bot token
2. Message your new bot anything (this opens the chat)
3. Get your chat id: open `https://api.telegram.org/bot<TOKEN>/getUpdates` in a browser and copy the `"chat":{"id":...}` number
4. Add both to a `.env` file in your workspace root:

```
TELEGRAM_BOT_TOKEN=your-token-here
TELEGRAM_CHAT_ID=your-chat-id-here
```

5. Tell Claude: "after writing my morning brief, send it to my Telegram." It sends via one call:

```bash
curl -s -X POST "https://api.telegram.org/bot$TELEGRAM_BOT_TOKEN/sendMessage" \
  -d chat_id="$TELEGRAM_CHAT_ID" \
  --data-urlencode text="$(cat outputs/briefs/$(date +%F).md)"
```

Telegram note: send the brief as plain text — Telegram shows markdown symbols literally, so the brief should use simple lines and spacing, not `#` headers, when pushed to your phone.

## Troubleshooting

- **Brief feels generic** → your `priorities.md` is too vague. The brief is only as specific as that file.
- **Telegram message never arrives** → re-check the token has no spaces, and that you messaged the bot first (bots can't message you until you start the chat).
- **Scheduled run didn't happen** → your machine was asleep. Schedule it for a time the laptop is awake, or run it manually with your coffee.
