---
name: viral-outlier-research
description: "Find the viral outliers in any Instagram niche. Give it 3-5 Instagram handles (or keep them in the config below), and it scrapes each account's recent reels via the Apify API, computes each account's median play count, scores every reel as a multiple of that median (2x+ = outlier, 5x+ = strong outlier), and writes a markdown report with per-account tables, cross-account patterns, and 5-10 idea starters based on the top outliers. Triggers: 'find outliers', 'viral research', 'what's blowing up in my niche', 'outlier research', 'research these accounts'."
---

# Viral Outlier Research

Scrape 3-5 Instagram accounts in your niche, find the reels that massively outperformed each account's own baseline, and turn them into content ideas. The multiple (plays ÷ that account's median plays) is the whole trick — it surfaces true breakouts on small accounts instead of just showing you big accounts' normal posts.

## Config — your niche accounts

Edit this list once, or override it by naming handles when you run the skill.

```
NICHE ACCOUNTS (3-5 handles, no @):
- handle_one
- handle_two
- handle_three
```

### FlowHQ niche groups

FlowHQ's research covers three niches. Each run uses ONE group (3-5 handles) so the per-account medians and cross-account patterns stay comparable. Replace the `REPLACE_ME_*` entries with real, public, actively-posting reel accounts before running — the skill must not run while any `REPLACE_ME_*` handle is present.

```
AI AUTOMATION CREATORS (workflows, agents, no-code/n8n/Make/Zapier-style automation):
- REPLACE_ME_automation_1
- REPLACE_ME_automation_2
- REPLACE_ME_automation_3

AI TOOLS CREATORS (tool reviews, "tools you're missing", prompt/tool walkthroughs):
- REPLACE_ME_tools_1
- REPLACE_ME_tools_2
- REPLACE_ME_tools_3

BUSINESS EDUCATION CREATORS (entrepreneurship, marketing, operations, founder lessons):
- REPLACE_ME_business_1
- REPLACE_ME_business_2
- REPLACE_ME_business_3
```

To run: "find outliers for the AI automation group" (or "AI tools" / "business education"). Naming handles in the request still overrides this config.

## When to use

- You want to know what's actually working in your niche right now, backed by real numbers.
- You're planning a content batch and want proven angles to adapt, not guesses.
- You found new accounts to watch and want a baseline read on them.

Not for: one-off deep dives on a single video, or platforms other than Instagram.

## Prerequisites

1. **Claude Code** — this skill runs inside it.
2. **An Apify account** — free tier includes monthly credit; this skill costs roughly **$0.05-0.10 per run**. Sign up at apify.com.
3. **Your Apify API key** — find it at **console.apify.com → Settings → Integrations → API tokens**. Copy the token.

## Setup (one-time)

Put the key in a `.env` file in your workspace root:

```
APIFY_API_KEY=apify_api_xxxxxxxxxxxxxxxx
```

If `.env` already exists, add the line to it. Make sure `.env` is in your `.gitignore`.

Then edit the Config section above with 3-5 handles from your niche. Pick accounts that post reels regularly — dead accounts give you nothing to measure.

## How it works

### Step 1 — Resolve handles

Use handles named in the request; otherwise use the Config list above. 3-5 accounts is the sweet spot. Strip any `@` or URL wrapping down to the bare handle.

### Step 2 — Scrape recent reels (one Apify call)

Load the key and call the Instagram scraper actor via REST — no MCP, no extra tools:

```bash
source .env
mkdir -p outputs/viral-research
curl -s --max-time 600 -X POST \
  "https://api.apify.com/v2/acts/apify~instagram-scraper/run-sync-get-dataset-items?token=$APIFY_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "directUrls": [
      "https://www.instagram.com/handle_one/",
      "https://www.instagram.com/handle_two/",
      "https://www.instagram.com/handle_three/"
    ],
    "resultsType": "reels",
    "resultsLimit": 60,
    "onlyPostsNewerThan": "6 months"
  }' > outputs/viral-research/raw.json
```

The input shape matters: `directUrls` takes full profile URLs, `resultsType: "reels"` returns reels with play counts, `resultsLimit: 60` applies per account, `onlyPostsNewerThan: "6 months"` keeps the baseline current. The call is synchronous and can take a few minutes for 5 accounts — the `--max-time 600` covers that.

Sanity-check the result before doing anything else: `raw.json` must be a non-empty JSON array. If it's empty, an error object, or missing accounts, go to Troubleshooting — do not proceed with partial guesses.

From each item, keep only these fields: `ownerUsername`, `shortCode`, `url`, `videoPlayCount`, `likesCount`, `commentsCount`, `timestamp`, `videoDuration`, `isPinned`, `caption`.

### Step 3 — Compute medians and multiples

Run this (inline python, stdlib only) rather than eyeballing numbers:

```bash
python3 - <<'EOF'
import json, statistics, collections
items = json.load(open("outputs/viral-research/raw.json"))
by_acct = collections.defaultdict(list)
for it in items:
    if it.get("videoPlayCount") is not None:
        by_acct[it["ownerUsername"]].append(it)
out = {}
for acct, reels in by_acct.items():
    med = statistics.median(r["videoPlayCount"] for r in reels)
    for r in reels:
        r["multiple"] = round(r["videoPlayCount"] / med, 1) if med else None
    reels.sort(key=lambda r: (r["multiple"] or 0), reverse=True)
    out[acct] = {"median": med, "count": len(reels), "reels": reels}
json.dump(out, open("outputs/viral-research/scored.json", "w"), indent=1, default=str)
for acct, d in out.items():
    print(f"{acct}: {d['count']} reels, median {d['median']:,.0f} plays")
EOF
```

Rules for scoring:
- **Multiple = plays ÷ that account's median.** 2x+ = outlier. 5x+ = strong outlier.
- **Pinned reels (`isPinned: true`) stay in the report but get a "pinned" flag** — pinned posts accumulate plays over time, so their multiple reads high. Judge them by the content, not the number.
- **Collapsed-median warning:** if an account's median is tiny relative to its follower count (most posts barely get plays), every multiple on that account is inflated. Flag the account in the report and lean on absolute play counts for it instead.

### Step 4 — Write the report

Write to `outputs/viral-research/YYYY-MM-DD-outliers.md` (today's date). Local markdown only — no uploads, no external tools.

**Every play count, multiple, and date in the report must trace to the actual API response in `raw.json`/`scored.json`. Never estimate, round up for effect, or fill a gap with a plausible number. If the scrape failed or an account came back empty, say exactly that in the report — a missing number reported honestly beats an invented one every time.**

### Step 5 — Hand it over

Tell the user the report path, the top 3 outliers across all accounts (link + multiple + one-line hook), and any warnings (empty accounts, collapsed medians, pinned inflation).

## Output format

```markdown
# Viral Outlier Report — YYYY-MM-DD
Accounts: @a, @b, @c · Window: last 6 months · Source: Apify instagram-scraper

## @handle_one — median 12,400 plays (58 reels)
| Reel | Plays | Multiple | Date | Length | Caption (first line) |
|---|---|---|---|---|---|
| [link](url) | 156,000 | 12.6x | 2026-05-12 | 34s | "I quit my job to..." |
| [link](url) | 41,000 | 3.3x 📌 pinned | 2026-02-01 | 61s | "The 5 tools I..." |
...top 10 by multiple, 2x+ only (fewer if fewer qualify)

⚠️ (only if it applies) Median collapsed vs followers — read absolute plays, not multiples.

## @handle_two — ...

## Cross-account patterns
3-6 bullets: hook styles, formats, topics, or lengths that repeat across the outliers.
Grounded in the captions and numbers above — cite which reels support each pattern.

## Idea starters
5-10 ideas, each: the outlier it's based on (link + multiple), the transferable
mechanic (why it worked), and a one-line adaptation for the user's own account.

## Data notes
Scraped via Apify apify~instagram-scraper on YYYY-MM-DD. Play counts = videoPlayCount
from the API response. Multiples = plays ÷ each account's own median. Any scrape
failures or empty accounts listed here.
```

Keep the writing plain. Patterns and ideas come from what's actually in the data — captions, lengths, timing, the numbers — not from generic content advice.

## Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| HTTP 401 / `Authentication token is not valid` | Bad or missing token | Re-copy the token from console.apify.com → Settings → Integrations → API tokens; check the `.env` line has no quotes or trailing spaces |
| Empty array `[]` or an account missing from results | Private account, typo in handle, or handle changed | Open `instagram.com/<handle>` in a browser to verify it exists and is public; fix the handle and re-run |
| HTTP 429 or actor errors about limits | Apify rate limit or out of credit | Check usage at console.apify.com; wait a few minutes or reduce to 3 accounts / `resultsLimit: 40` |
| Call times out | Sync run exceeded the wait | Re-run with fewer accounts, or raise `--max-time`; results for a timed-out run are still visible at console.apify.com → Actors → Runs |
| `videoPlayCount` missing on items | Result set includes non-reel posts | Confirm `resultsType` is exactly `"reels"`; skip items without the field and note the skips in Data notes |

If a run fails partway, the report still gets written — with the accounts that succeeded, and an honest note about the ones that didn't.
