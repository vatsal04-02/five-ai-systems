---
name: life-wiki
description: Maintain a personal knowledge base that the AI writes and the human reads. Three jobs — ingest raw captures from wiki/raw/ into linked wiki pages, answer questions from the wiki with citations, and run a monthly health check. Use when the user says "ingest", "process my inbox", "add this to my wiki", asks a question about their own notes/people/projects ("what does my wiki say about X"), or asks for a wiki health check.
---

# Life Wiki

A personal knowledge base kept in the `wiki/` folder of this workspace. The human dumps anything into `wiki/raw/` — links, notes, voice-memo transcripts, descriptions of screenshots. The AI turns those dumps into organized, cross-linked pages. The human never has to file anything.

Full conventions live in `wiki-schema.md` (next to this file, or copied into `wiki/schema.md` during setup). Read it before your first ingest of a session. No API keys, no external services — this whole system is files in a folder. Obsidian (free) is the nicest way to browse it, but any file explorer works.

Three jobs: **ingest**, **query**, **health check**.

---

## Job 1: Ingest

Trigger: "ingest", "process my inbox", "add this to the wiki", or new files sitting in `wiki/raw/`.

1. **Find unprocessed sources.** List `wiki/raw/` and compare against `wiki/log.md`. Anything not logged is unprocessed. If the user named a specific file or pasted content directly, use that.
2. **Read the source in full.** Decide what it is (article, brain dump, note about a person, project update, research material) and which area(s) of the wiki it touches. One source can touch several areas.
3. **Confirm direction (skip in batch mode).** Give the user 3-5 key takeaways and ask if they want to emphasize or redirect anything. If they said "process everything", skip this and just work.
4. **Read the current wiki state.** Read `wiki/index.md`, then open any existing pages this source will update or connect to. Never create a duplicate of a page that already exists under a slightly different name.
5. **Create or update pages.** For each concept, person, project, or idea in the source:
   - Page exists → update it, refresh `updated:` in frontmatter, cite the source.
   - No page → create one in the right folder with full frontmatter (see schema).
   - A meaty source should touch roughly 3-10 pages. If you only touched one, you probably under-extracted. If the source is trivial ("dentist Tuesday"), it's not wiki material — say so and move on.
6. **Cross-link.** Add `[[wiki links]]` between every related page, including across folders. A person page links to the projects they're part of; a research page links to the beliefs it challenged.
7. **Flag contradictions.** If new info conflicts with an existing page, don't silently overwrite. Add a visible block:
   > **CONTRADICTION:** [old claim] vs [new claim] from [source]
8. **Update `wiki/index.md`** with any new pages (alphabetical within each section, one-line description each).
9. **Log it.** Append an entry to `wiki/log.md` (format in the schema).
10. **Report.** Tell the user: pages created, pages updated, links made, contradictions flagged, and one gap you noticed ("you mention X a lot but there's no page for it").
11. **Clear the inbox.** Move processed files from `wiki/raw/` into `wiki/raw/processed/` (create it if missing). Never edit or delete a raw file's content — raw captures are the record.

### Ingest rules

- Raw files are read-only. Archive them after processing; never rewrite them.
- Every claim on a wiki page cites where it came from: `[Source: filename or link]`.
- Cross-link generously — the value of the wiki is the connections, not the pages.
- One well-connected page beats five orphans.
- Person pages: filename `firstname-lastname.md`, covering who they are, how the user knows them, key interactions, what the user has learned from them.

---

## Job 2: Query

Trigger: any question the wiki could answer — "what does my wiki say about X", "what do I know about Y", "when did I decide Z".

1. **Read `wiki/index.md`** to find candidate pages.
2. **Read every relevant page**, across folders. The best answers connect areas — a work question often has a personal-growth thread.
3. **Answer directly**, with `[Source: page-name]` after each claim. If the wiki can't support part of the answer, say exactly that: "The wiki doesn't cover X yet." Never pad a wiki answer with general knowledge without flagging it as outside the wiki.
4. **Offer to file back.** If the answer surfaced a connection worth keeping, offer to save it as a new page or an update to an existing one. If the user agrees, do it and log it.

### Query modes

- **`gaps`** — read the whole index, name the 5 biggest holes: topics mentioned but never explained, people without pages, thin areas, questions the wiki implies but can't answer. Suggest what to capture next.
- **`connections A B`** — trace every path between two topics through the link graph. Surface the non-obvious ones.
- **`brief [topic]`** — a ~500-word briefing built only from wiki content: current understanding, key tensions, open questions, next steps. Cited throughout.

---

## Job 3: Monthly Health Check

Trigger: "wiki health check", "lint the wiki", or roughly once a month.

Scan the whole wiki for:

1. **Contradictions** between pages (including unresolved CONTRADICTION blocks).
2. **Stale claims** — pages not touched in 90+ days making time-sensitive claims.
3. **Orphan pages** — no inbound `[[links]]` from anywhere.
4. **Ghost concepts** — `[[links]]` pointing at pages that don't exist yet.
5. **Missing citations** — claims with no `[Source: ...]`.
6. **Index drift** — pages missing from `wiki/index.md`, or index entries pointing at deleted pages.

Report findings in chat grouped by severity (fix now / fix soon / cosmetic), fix the mechanical ones directly (index drift, dead links) with the user's go-ahead, and log the check in `wiki/log.md`.

---

## Standing rules

- Every page: YAML frontmatter with `tags: []`. No exceptions.
- Every cross-reference: `[[wiki link]]` format.
- `wiki/index.md` lists every page. `wiki/log.md` records every ingest, filed-back answer, and health check, newest first.
- The AI writes, the human reads. Don't ask the user to file, tag, or organize anything — that's your job.
