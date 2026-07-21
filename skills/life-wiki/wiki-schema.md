# Wiki Schema

The rules that keep the wiki consistent. The AI follows these on every ingest, query, and health check. The human never has to think about them.

This is a plain-files system: markdown in folders, nothing else. No API keys, no external services, no cost. Obsidian (free) renders the `[[links]]` as a clickable graph and is the nicest way to browse — but the wiki works fine as ordinary files.

## Folder Layout

```
wiki/
  raw/            Universal inbox. Dump anything here — links, notes,
                  voice-memo transcripts, screenshot descriptions.
                  The AI reads these but never edits them.
  raw/processed/  Where raw files go after ingest. The permanent record.
  personal/       People, growth, experiences, beliefs, health, life.
  work/           Career, business, projects, decisions, money.
  research/       Topics being actively studied or explored.
  index.md        Master directory of every wiki page.
  log.md          Chronological record of every ingest, update, and check.
  schema.md       This file.
```

Add or rename topic folders to fit the user's life (set during onboarding). `raw/`, `index.md`, and `log.md` are fixed.

## Page Frontmatter

Every wiki page starts with YAML frontmatter. `tags` is mandatory on every file, always.

```yaml
---
title: Topic Name
area: personal | work | research
created: YYYY-MM-DD
updated: YYYY-MM-DD
status: draft | reviewed | needs_update
tags: [wiki, area-tag, topic-tags]
---
```

After the frontmatter:

- Open with a one-paragraph summary of the page.
- Body in short sections. Facts over prose.
- Every factual claim cites its origin: `[Source: filename.md]` or `[Source: URL]`.
- When new info conflicts with what's already on the page, keep both visibly:

  > **CONTRADICTION:** [old claim] vs [new claim] from [source]

## Linking Rules

- Every cross-reference between pages uses `[[wiki link]]` format: `[[marathon-training]]`, `[[jane-doe]]`.
- Filenames are lowercase-with-hyphens; the link matches the filename without `.md`.
- Link across folders freely — a person page links to projects, a research page links to beliefs it changed.
- A page with zero inbound links is an orphan. Orphans get flagged in the monthly health check.
- Linking to a page that doesn't exist yet is allowed — it marks a page worth creating.

## index.md

The master directory. One section per folder, entries alphabetical, one line each:

```markdown
# Wiki Index

## Personal
- [[jane-doe]] — college friend, now product designer at Acme
- [[running]] — training history, injuries, race results

## Work
- [[acme-project]] — client project, kicked off March 2026

## Research
- [[spaced-repetition]] — how memory systems actually work
```

Every page appears in the index. A page missing from the index is invisible — the AI reads the index first on every query.

## log.md

Newest entries at the top. One entry per ingest, filed-back answer, or health check:

```markdown
## YYYY-MM-DD — ingest: voice-note-tuesday.md
Created 2 pages ([[jane-doe]], [[career-pivot]]), updated 3.
Areas: personal, work. Flagged 1 contradiction on [[career-pivot]].

## YYYY-MM-DD — health check
2 orphans linked, index synced, no contradictions.
```

The log is how the AI knows what's already been processed — a raw file with no log entry is unprocessed.

## What Belongs in the Wiki

- People, projects, decisions, beliefs, lessons, topics under study.
- Anything the user will want to ask about in six months.

What doesn't: throwaway to-dos ("book dentist"), duplicates of things already captured, and raw sources themselves (those stay in `raw/processed/`). When in doubt, extract the durable part and skip the rest.

## Division of Labor

The AI: routes, files, tags, links, indexes, logs, flags contradictions.
The human: dumps into `raw/`, asks questions, reads pages, makes the calls when contradictions surface.
