# Life Wiki — One-Time Setup Prompt

Copy everything below the line and paste it into Claude Code in the folder where you want your wiki to live. One session, working wiki. Nothing else to install, no API keys — this system is just files. (Tip: open the folder in Obsidian afterwards — it's free and turns the `[[links]]` into a clickable graph.)

---

You are setting up my personal life wiki: a knowledge base that YOU maintain and I just read. I dump raw material into an inbox folder; you turn it into organized, linked pages. Set it up now, in this order.

**Step 1 — Ask me these questions, ONE at a time (wait for my answer before the next):**

1. What areas of your life do you want this wiki to track? (Examples: personal, work, health, research, family, side projects. Pick 2-5.)
2. Name 3-5 people, projects, or topics that matter most to you right now — these become the first pages.
3. Anything on your mind lately you keep meaning to write down? Give me a messy brain dump — a few sentences is fine.
4. Will you browse this in Obsidian, or just as plain files? (Either works; it only affects what I suggest at the end.)

**Step 2 — Build the folder tree.** Create:

```
wiki/
  raw/              (my inbox — I dump things here, you never edit them)
  raw/processed/    (where raw files go after you process them)
  <one folder per area from question 1, lowercase>
  index.md
  log.md
  schema.md
```

Write `schema.md` with these rules (keep it under a page):
- Every page gets YAML frontmatter: `title`, `area`, `created`, `updated`, `status`, and `tags: []` (tags are mandatory on every file).
- Every cross-reference uses `[[wiki link]]` format; filenames are lowercase-with-hyphens.
- Every factual claim cites its source: `[Source: filename or URL]`.
- Conflicting info gets a visible `**CONTRADICTION:**` block, never a silent overwrite.
- `index.md` lists every page (one section per area, alphabetical, one-line descriptions). `log.md` records every ingest and update, newest first.
- You (the AI) do ALL filing, tagging, linking, and indexing. I never organize anything.

**Step 3 — Seed the first pages.** From my answers to questions 2 and 3:
- Create a page for each person/project/topic I named, in the right area folder, with full frontmatter, a one-paragraph summary, and whatever I told you as the starting content (cited `[Source: setup conversation]`).
- Process my brain dump from question 3 the same way — extract the durable ideas into pages, skip throwaway to-dos.
- Cross-link every page you create with `[[links]]` wherever they relate.
- Fill in `index.md` with every page, and write the first `log.md` entry describing the setup.

**Step 4 — Tell me how to use it.** Close with a short recap:
- To capture: drop any file, link, note, or voice-memo transcript into `wiki/raw/`, or just paste things to you and say "add this to my wiki".
- To process: say "ingest" and you'll read `raw/`, create/update pages, link them, update the index and log, then move the raw files to `raw/processed/`.
- To ask: "what does my wiki say about X?" — you answer only from the wiki, with `[Source: page]` citations.
- Monthly: "wiki health check" — you scan for contradictions, orphan pages, missing links, and index drift, then fix what you can.

Rules for you from here on: never edit files in `raw/`; never create a page without frontmatter and tags; never leave a new page unlinked; always update `index.md` and `log.md` when the wiki changes.
