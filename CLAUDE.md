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
  sources/          one page per raw item; the citation targets
  *.md              concept and entity pages, flat
  catalog.base      Obsidian table view over the pages (for humans, not for you)
assets/             images
```

`wiki/` is flat on purpose. Obsidian resolves `[[mcp]]` no matter which folder the page
sits in, so folders buy nothing the `category:` and `type:` fields don't already provide —
and a field is cheap to change where a directory tree is not. A page that turns out to be
two topics gets recategorized with one edit. Do not create `concepts/`, `entities/`, or
`analyses/` subfolders. `sources/` is separate only because those pages are 1:1 with
`raw/`.

There is no `log.md` and no `index.md`. Git is the log (see [Commits](#commits)),
and the catalog is derived rather than stored (see [Finding pages](#finding-pages)).

## Page format

Frontmatter on every page in `wiki/`. Minimal — git owns dates, so no `created`/`updated`:

```yaml
---
type: page
tags: [memory, context]
category: context-memory
summary: "One line saying what this page is about, in quotes."
---
```

Source pages add where the material came from:

```yaml
---
type: source
tags: [memory, context]
category: context-memory
summary: "One line saying what this source argues."
raw: raw/2026-09-09-karpathy-llm-wiki-gist.md
url: https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f
---
```

`type` is `page` or `source`.

`summary` is the retrieval hook — it is what a future session reads to decide whether to
open this page, so write it to answer *what questions does this page settle?* rather than
to describe the page. One line, always double-quoted (summaries tend to contain colons).

`category` and `tags` are lowercase and hyphenated, and both are **reused, not invented**.
There is no fixed list of either; the live set is whatever the pages currently use:

```bash
head -n 12 wiki/*.md wiki/sources/*.md | grep '^category:' | sort -u
head -n 12 wiki/*.md wiki/sources/*.md | grep '^tags:'     | sort -u
```

(`head -n 12` is the frontmatter window. Grepping whole files also matches the YAML
examples quoted inside page bodies, which is how you get phantom categories.)

Check that before adding a new value. A new category is fine when nothing fits — just
make it a deliberate choice rather than a synonym for one that already exists.

Body conventions:

- `# Title` as the first line, matching the filename.
- Filenames are lowercase and hyphenated: `tool-use.md`, `context-window.md`.
- Link generously with `[[wikilinks]]`. A link to a page that doesn't exist yet is a
  legitimate marker of a gap — it shows up in Obsidian's Unresolved links pane.
- Concept pages state claims and cite the source page the claim came from:
  `... one page revised by many sources ([[karpathy-llm-wiki-gist]]).`
- When two sources disagree, say so on the page. Don't silently pick a winner.

## Finding pages

There is no stored index. The catalog is **derived** from the pages themselves, so it
cannot go stale and nothing has to be kept in sync:

```bash
head -n 12 wiki/*.md wiki/sources/*.md | grep -E '^(==>|category:|summary:)'
```

That is the first command of most sessions. It prints every page with its category and
one-line summary — the same thing a hand-written index would, minus the drift.

Narrow it when you already know the shape of the question:

```bash
grep -rl --include='*.md' '^category: tool-use-protocols' wiki/   # one category
grep -rli --include='*.md' 'speculative decoding' wiki/           # full text
grep -rl --include='*.md' '\[\[mcp\]\]' wiki/                      # what links here
```

Use the derived catalog to *choose* pages and grep to *catch what it missed*. A summary
is one line and will not mention everything a page covers, so when a question doesn't map
cleanly onto a summary, grep the full text before concluding the wiki has no answer.

`wiki/catalog.base` renders the same information as a sortable table inside Obsidian.
That view is for the human — a `.base` file is a query definition, so reading it gives
you the query, not the results. Use the command above instead.

## Workflows

### Ingest

One raw item at a time. The cost is paid once here so that later queries read two pages
instead of re-reading the source.

1. Run the catalog command (see [Finding pages](#finding-pages)) to see what exists.
2. Read the raw item. This is the **only** time it gets read — everything downstream
   reads the wiki instead.
3. Discuss the key takeaways with the user before writing. Let them tell you what to
   emphasize.
4. Write `wiki/sources/<slug>.md` — roughly one page, `type: source`, with `category:`,
   `summary:`, `raw:` and `url:`.
5. Create or revise **every** wiki page the source touches. This is the part that makes
   the wiki compound: a new source on MCP doesn't just get a source page, it revises
   `tool-use.md` too. Add links in both directions — the new page links to what it
   builds on, and the existing pages link forward to it.
6. Give every new page a `category:` and a `summary:`, reusing existing values where
   they fit. This is what puts the page into the catalog — there is no index to update.
7. Commit.

A single source touching four or five pages is normal. One that touches only its own
source page usually means step 5 was skipped.

### Query

1. Run the catalog command first, and grep when the question doesn't map cleanly onto a
   summary. See [Finding pages](#finding-pages).
2. Open the pages that survive that, and only those.
3. Answer with citations back to `wiki/sources/` pages.
4. If the answer is worth keeping — a comparison, a synthesis, a connection nobody had
   written down — file it as a new wiki page with its own `category:` and `summary:`.
   Explorations should compound the same way ingested sources do.

Do not read `raw/` to answer a query. If the wiki can't answer it, that is a finding:
say so, and propose either a lint pass or a new source.

### Lint

Gather the mechanical facts by command first — they're exact and cheap, and they narrow
where the expensive reading has to happen:

```bash
# sources that were never ingested
ls raw/ && ls wiki/sources/

# pages missing a field that makes them findable
for f in wiki/*.md wiki/sources/*.md; do
  head -n 12 "$f" | grep -q '^summary:'  || echo "no summary:  $f"
  head -n 12 "$f" | grep -q '^category:' || echo "no category: $f"
done

# categories used only once — often a synonym of an existing one
head -n 12 wiki/*.md wiki/sources/*.md | grep '^category:' | sort | uniq -c | sort -n

# staleness, per page
git log -1 --format=%cd -- <page>
```

A page missing `category:` or `summary:` is invisible to the catalog — that is the one
failure mode the derived approach has, and it is exactly checkable, which a stale
hand-written index never was. Categories with a count of 1 are worth a look: often a
synonym of an existing one rather than a genuinely new bucket.

Broken `[[links]]` come from Obsidian's Unresolved links pane, which is authoritative;
don't reimplement it with grep.

Then do the pass only an LLM can do — read the pages and look for:

- pages that contradict each other
- claims a newer source has superseded
- concepts recurring across several sources that still have no page of their own
- pages that have grown to cover two ideas and should be split
- pages that ought to link to each other and don't
- summaries that no longer match what the page grew into

Report findings, propose the fixes, and apply them only once the user approves.

## Commits

The commit log replaces `log.md`, and it's better: it records what actually changed
rather than what I claim changed. Subject is a prefix plus the source or question;
body lists the touched pages.

```
ingest: MCP specification

wiki/sources/mcp-spec.md, wiki/mcp.md, wiki/tool-use.md
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
- Every page in `wiki/` has frontmatter, including `category:` and `summary:` — those
  two fields are what makes it findable at all.
- New raw files are named `YYYY-MM-DD-<slug>.<ext>`.
