# round-vault

An LLM-maintained wiki on **LLM-agent-native systems** — how systems are designed when
an LLM agent, not a human, is the primary operator.

You write and maintain this wiki. The user curates sources, asks questions, and decides
what matters. Do not ask the user to write pages; that is your job.

Everything here is in **English**.

## Layout

```
raw/                sources, verbatim and immutable — you never edit these
wiki/
  index.md          catalog: category -> page -> one-line summary
  sources/          one page per raw item; the citation targets
  *.md              concept and entity pages, flat
assets/             images
```

`wiki/` is flat on purpose. Obsidian resolves `[[mcp]]` no matter which folder the page
sits in, so folders buy nothing that `index.md` and the `type:` field don't already
provide — and grouping is cheap to change there and expensive to change in the
filesystem. Do not create `concepts/`, `entities/`, or `analyses/` subfolders.
`sources/` is separate only because those pages are 1:1 with `raw/`.

There is no `log.md`. Git is the log; see [Commits](#commits).

## Page format

Frontmatter on every page in `wiki/`. Minimal — git owns dates, so no `created`/`updated`:

```yaml
---
type: page
tags: [memory, context]
---
```

Source pages add where the material came from:

```yaml
---
type: source
tags: [memory, context]
raw: raw/2026-09-09-karpathy-llm-wiki-gist.md
url: https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f
---
```

`type` is `page` or `source`. Tags are lowercase, hyphenated, and reused across pages —
check `index.md` for existing tags before inventing one.

Body conventions:

- `# Title` as the first line, matching the filename.
- Filenames are lowercase and hyphenated: `tool-use.md`, `context-window.md`.
- Link generously with `[[wikilinks]]`. A link to a page that doesn't exist yet is a
  legitimate marker of a gap — it shows up in Obsidian's Unresolved links pane.
- Concept pages state claims and cite the source page the claim came from:
  `... one page revised by many sources ([[karpathy-llm-wiki-gist]]).`
- When two sources disagree, say so on the page. Don't silently pick a winner.

## Workflows

### Ingest

One raw item at a time. The cost is paid once here so that later queries read two pages
instead of re-reading the source.

1. Read `wiki/index.md` to see what already exists.
2. Read the raw item. This is the **only** time it gets read — everything downstream
   reads the wiki instead.
3. Discuss the key takeaways with the user before writing. Let them tell you what to
   emphasize.
4. Write `wiki/sources/<slug>.md` — roughly one page, `type: source`, with `raw:` and
   `url:`.
5. Create or revise **every** wiki page the source touches. This is the part that makes
   the wiki compound: a new source on MCP doesn't just get a source page, it revises
   `tool-use.md` too. Add links in both directions — the new page links to what it
   builds on, and the existing pages link forward to it.
6. Add a row to `index.md`.
7. Commit.

A single source touching four or five pages is normal. One that touches only its own
source page usually means step 5 was skipped.

### Query

1. Read `index.md` first. It is the retrieval layer — this is what the wiki uses
   instead of a vector database.
2. Open only the pages the index points at.
3. Answer with citations back to `wiki/sources/` pages.
4. If the answer is worth keeping — a comparison, a synthesis, a connection nobody had
   written down — file it as a new wiki page and index it. Explorations should compound
   the same way ingested sources do.

Do not read `raw/` to answer a query. If the wiki can't answer it, that is a finding:
say so, and propose either a lint pass or a new source.

### Lint

Gather the mechanical facts by command first — they're exact and cheap, and they narrow
where the expensive reading has to happen:

```bash
ls raw/ && ls wiki/sources/           # sources that were never ingested
git log -1 --format=%cd -- <page>     # staleness, per page
```

Broken `[[links]]` come from Obsidian's Unresolved links pane, which is authoritative;
don't reimplement it with grep.

Then do the pass only an LLM can do — read the pages and look for:

- pages that contradict each other
- claims a newer source has superseded
- concepts recurring across several sources that still have no page of their own
- pages that have grown to cover two ideas and should be split
- pages that ought to link to each other and don't

Report findings, propose the fixes, and apply them only once the user approves.

## Commits

The commit log replaces `log.md`, and it's better: it records what actually changed
rather than what I claim changed. Subject is a prefix plus the source or question;
body lists the touched pages.

```
ingest: MCP specification

wiki/sources/mcp-spec.md, wiki/mcp.md, wiki/tool-use.md, wiki/index.md
```

Prefixes: `ingest:`, `query:`, `lint:`, and `note:` for everything else — manual saves
from Obsidian's git plugin, schema edits, repo maintenance.

Recovering history:

```bash
git log --oneline --grep '^ingest:' | head    # recent activity
git show --stat <sha>                         # what one ingest touched
git log -S'<claim text>' -- wiki/             # when a claim entered the wiki
```

## Invariants

- **Never edit anything under `raw/`.** Immutability is what keeps the wiki
  recompilable, verifiable, and auditable — ground truth must never blur with
  interpretation. If a source is wrong, say so on the wiki page, don't fix the source.
- Everything is written in **English**.
- Every page in `wiki/` has frontmatter.
- Every new page gets a row in `index.md` in the same commit.
- New raw files are named `YYYY-MM-DD-<slug>.<ext>`.
