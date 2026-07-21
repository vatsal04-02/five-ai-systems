# The 5 AI Systems

Five AI systems that run parts of my life and business — a morning brief that hits my phone before I wake up, a viral research engine with real numbers, a full audit of my own content, a wiki of my entire life that an AI maintains, and a diagram builder for my videos.

This folder is all five, ready to install. No coding. If you can copy-paste, you can run everything here.

---

## What you need

**The one real requirement: Claude Code.**
It's Anthropic's AI that works inside a folder on your computer — reading files, writing files, running the systems. Get it at [claude.com/claude-code](https://claude.com/claude-code). It needs a Claude subscription (Pro, ~$20/month, is enough).

Everything else is optional and depends on which systems you run:

| System | What it needs | Cost |
|---|---|---|
| Life Wiki | Nothing (Obsidian optional, free) | Free |
| Morning Brief | Nothing (Telegram optional, free) | Free |
| Diagram Builder | Nothing (open results at excalidraw.com, free) | Free |
| Viral Outlier Research | Apify account + API key | ~$0.05–0.10 per run (free credit covers your first runs) |
| Content Audit | Same Apify key | ~$0.10–0.30 per audit |

Total to run all five: a Claude subscription plus pocket change on Apify.

## Install (10 minutes)

1. **Get Claude Code** (link above) and make sure it runs — open a terminal, type `claude`, say hi.
2. **Put this folder somewhere permanent** (Documents is fine). It becomes your AI workspace — the systems write their outputs inside it.
3. **Open a terminal in this folder and start Claude Code.** It can see all five skills in `skills/` from here.
4. **Only if you're using Viral Research or Content Audit:** copy `.env.example` to `.env` and add your Apify key (walkthrough below).
5. Start with one system — the order below is the one I'd follow.

## The order I'd install them in

**Day 1 — Life Wiki** (free, 20 minutes). Open `skills/life-wiki/SETUP-PROMPT.md`, copy the prompt, paste it into Claude Code, answer its four questions. You'll have a working wiki in one session. Everything else gets better once this exists, because the other systems can read it.

**Day 2 — Morning Brief** (free, 5 minutes). Fill in the small priorities file (`skills/morning-brief/SKILL.md` shows the template), then say "run my morning brief" tomorrow with your coffee. When you're ready for the phone-notification version, the skill walks you through the free Telegram setup — that's the one from the video.

**Week 1 — Content Audit** (needs the Apify key). Say "audit my Instagram" and give your handle. Ten minutes later you know which of your content actually works, by the numbers — most people have never seen their own account like this.

**Week 1 — Viral Outlier Research** (same key). Add 3–5 accounts from your niche to the config in the skill file, then say "find outliers". You get the reels that massively beat each account's own baseline — real play counts, not vibes.

**Whenever — Diagram Builder** (free). Say "diagram my content funnel" or any process you want visualized. Open the file it makes at excalidraw.com and drag things around.

## Getting your Apify key (for Viral Research + Content Audit)

Apify is a scraping platform — it's how the AI pulls real Instagram numbers legally through an API instead of guessing.

1. Sign up free at [apify.com](https://apify.com) (free tier includes monthly credit — your first runs cost nothing)
2. Go to **console.apify.com → Settings → Integrations → API tokens**
3. Copy the token
4. In this folder: copy `.env.example` to a file named `.env`, and paste your token after `APIFY_API_KEY=`
5. Never share that file or commit it anywhere public

That one key powers both systems.

## What each system actually does

**Morning Brief** — `skills/morning-brief/` — Reads your priorities file and yesterday's brief, writes today's: one focus, carried-over loops, one writing prompt built from what you're actually working on. Level 3 sends it to your phone via a free Telegram bot before you wake up.

**Viral Outlier Research** — `skills/viral-outlier-research/` — Scrapes recent reels from accounts you pick, computes each account's median plays, and scores every reel as a multiple of that median. A 12x reel on a small account is a format that works; this finds those instead of just showing you big accounts being big.

**Content Audit** — `skills/content-audit/` — Same math pointed at YOUR account, 12 months deep. Groups your content into lanes, shows which lane is your reach engine, which posts stalled and why, and ends with a playbook specific to your numbers.

**Life Wiki** — `skills/life-wiki/` — A knowledge base the AI maintains and you just read. Dump anything into the inbox folder (notes, links, voice-memo transcripts); say "ingest" and it becomes organized, linked pages. Ask it anything about your own life and get cited answers. Zero cost, and honestly the one that changes how you work the most.

**Diagram Builder** — `skills/diagram-builder/` — Describe any concept, process, or system and get a polished dark-mode diagram as an editable file. Free path needs no keys at all.

## FAQ

**Do I need to know how to code?** No. Every system runs by talking to Claude Code in plain English. The one technical moment is pasting an API key into a text file, and the walkthrough above covers it.

**Where does my data go?** Everything lives in this folder on your machine. The only things that leave: Apify API calls (public Instagram data in, numbers out) and, if you enable it, the Telegram message to your own phone.

**Something broke.** Every skill has a Troubleshooting section at the bottom covering the failures we actually hit building these. Start there — then just paste the error to Claude Code and ask.

**Can I change the systems?** They're plain text files. Edit anything. That's the point.
