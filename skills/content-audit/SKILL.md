---
name: content-audit
description: Audit your own Instagram account from 12 months of real reel data. Scrapes your reels via Apify, computes your account's median plays, finds which content lanes actually drive reach, deep-analyzes your top outliers and a control group of stalled posts, and ends with a personalized playbook. Use when the user says "audit my account", "audit my Instagram", "what's working on my account", "content audit", or gives their own IG handle and asks what to post more of. Does NOT trigger for auditing competitors or writing new content.
---

# Content Audit

Find out what your audience actually rewards — from your own data, not guesses. This skill pulls your last 12 months of Instagram reels, measures every post against your account's median, and tells you which content lane is your reach engine, which posts overperformed and why, and what to double down on.

## When to use

- You've been posting for months and don't know what's actually working
- You're planning next quarter's content and want data, not vibes
- You're coming back from a posting break and need to know what to lead with
- You suspect one type of content carries your account but can't prove it

You need an account with reels published in the last 12 months. 20+ reels gives a solid read; under 10 makes medians shaky (the skill will say so).

## Prerequisites

1. **Claude Code** installed and running in a workspace folder
2. **An Apify account** — [apify.com](https://apify.com), free tier includes monthly credit. A 100-reel audit costs roughly $0.10–0.30 in Apify usage.
3. **Your Apify API key** — Apify Console → Settings → API & Integrations → copy your personal API token

## Setup

Create a `.env` file in your workspace root (or add to the existing one):

```
APIFY_API_KEY=apify_api_xxxxxxxxxxxx
```

That's the only credential this skill needs. Never paste the key into chat or commit it to git — add `.env` to `.gitignore`.

## How it works

### Step 1 — Get the handle

Ask the user for their Instagram handle if not given. Confirm it's THEIR account — this audit reads captions and judges posts against the account's own baseline, so it only makes sense on an account the user owns or runs.

### Step 2 — Scrape 12 months of reels

Load the key and call the Apify Instagram scraper directly via REST (synchronous run, returns dataset items):

```bash
source .env
curl -s -X POST \
  "https://api.apify.com/v2/acts/apify~instagram-scraper/run-sync-get-dataset-items?token=$APIFY_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "directUrls": ["https://www.instagram.com/HANDLE/"],
    "resultsType": "reels",
    "resultsLimit": 100,
    "onlyPostsNewerThan": "12 months"
  }' > raw-reels.json
```

The run takes 1–3 minutes. From each item, keep only these fields (drop the rest — responses are large):

`shortCode`, `url`, `videoPlayCount`, `likesCount`, `commentsCount`, `timestamp`, `videoDuration`, `isPinned`, `caption`

Write the trimmed dataset to a working file. Sanity-check it: if the response is empty or an error object, stop and go to Troubleshooting — do not fabricate data.

### Step 3 — Compute the baseline

- **Field** = all scraped reels from the 12-month window (keep duplicates and broken low-play uploads — a median shrugs them off; note their presence).
- **Account median** = median of `videoPlayCount` across the field.
- **Multiple** = each reel's plays ÷ the account median. This is the core metric: 1.0x = typical post, 5x+ = outlier.
- **Pinned posts**: flag every `isPinned: true` reel. Pinned posts get inflated views from profile visits — include them in the field count but exclude them from outlier and engagement conclusions, and say so in the report.

### Step 4 — Cluster into content lanes

Read every caption and group the reels into **3–6 content lanes by topic**. The lane names come from what this account actually posts — infer them (e.g. "gear demos", "client stories", "how-to systems", "personal/mindset"), don't force a preset taxonomy. Every reel gets exactly one lane. For each lane compute: post count, median plays, best post.

This is the heart of the audit: reach lives in lanes, and most creators over-post their weakest lane.

### Step 5 — Deep-read the outliers and the control group

- **Top ~8 outliers** by multiple (excluding pinned). For each: verbatim hook from the caption's first line, lane, duration, plays, multiple, and what the caption reveals about format (listicle, before/after, demo, story, giveaway, comment-gate CTA).
- **Control group: 2–3 underperformers** (at or below 1x) that the user clearly put effort into. Say plainly why each likely stalled — hook with no stake, explaining instead of showing, no topic anchor up front.
- **Patterns**: distill 3–5 patterns the winners share and the stalled posts lack. Ground every pattern in specific posts — "6 of the top 8 open with a number in the first line" beats "strong hooks work".

### Step 6 — Honesty checks (non-negotiable)

- Every number in the report must trace to the API response. No estimates, no invented benchmarks, no industry averages.
- Flag partner-boosted-looking posts: huge plays with a tiny like ratio (roughly under 0.5% likes-to-plays) reads as paid or boosted reach — keep the row, exclude it from pattern conclusions, and note why.
- Flag incentivized engagement (giveaways, comment-gates) — comments there measure the mechanic, not the content.
- If the field is under ~10 reels, state that medians are weak and soften conclusions accordingly.
- If the scrape fails or the account is private, report that plainly and stop. Never fill gaps with plausible-sounding numbers.

### Step 7 — Write the report

Save to `outputs/content-audit/YYYY-MM-DD-audit.md` (create the folder if needed). Local markdown only — no uploads, no external docs. Show the user the file path and the top-line verdict.

## Report structure

Mirror this exact section order:

```markdown
# IG Reel Audit — @handle (12 months)

> One-line headline stat (e.g. "10% of posts earned 50% of views — all in one lane.")
> Field: N reels, date range · total plays · median X plays · counts as of [scrape date]

## The verdict
2–3 numbered findings. Lead with the biggest one. Each backed by numbers from the data.

## Lane medians
| Lane | Posts | Median plays | Best |

## The top N, deep-analyzed
| # | Reel (linked) | Plays | x median | Lane | Verbatim hook |
Then per-post notes: what the format is, why it likely won, any caveats
(boosted-looking reach, incentivized comments).

## Why they win — patterns
3–5 numbered patterns, each grounded in specific posts and counts.

## Control group — posts that stalled
2–3 underperformers with plays, multiple, and a plain-language read on why.

## Playbook
Checklist of 4–6 personalized actions: the comeback/next post (name the format
and the lane), the weekly mix (which lane for reach, which for trust, which
for leads), what to reframe, what proven winner to re-run.

## Data notes
Scrape source and date, field definition, what was excluded and why (pinned,
boosted-looking, incentivized), how multiples were computed.
```

Tone: verdict first, numbers everywhere, no hedging. The user should finish reading and know exactly what to post next.

## Troubleshooting

- **Empty result / `[]`** — Usually a typo'd handle or a private account. Verify the URL loads in a browser logged out. Private accounts cannot be scraped; say so and stop.
- **401 / "invalid token"** — `APIFY_API_KEY` missing or wrong in `.env`. Re-copy from Apify Console → Settings → API & Integrations.
- **Timeout on run-sync** — Large accounts can exceed the sync window. Re-run with `"resultsLimit": 60`, or use the async pattern: POST to `/v2/acts/apify~instagram-scraper/runs?token=...`, poll the run, then GET the dataset items.
- **`videoPlayCount` is null on some items** — Some items are photo posts or older uploads. Drop items without a play count from the field and note the count dropped.
- **Fewer reels than expected** — `onlyPostsNewerThan: "12 months"` only returns what exists; the account may simply have posted less. Report the real field size, don't pad it.
- **Costs concern** — Each run is metered by results. 100 reels ≈ $0.10–0.30. Check usage in the Apify Console under Billing.
