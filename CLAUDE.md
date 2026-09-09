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
  index.md          the map: one row per topic area, NOT per page
  sources/          one page per raw item; the citation targets
  *.md              concept and entity pages, flat
assets/             images
```

`wiki/` is flat on purpose. Obsidian resolves `[[mcp]]` no matter which folder the page
sits in, so folders buy nothing the `category:` and `type:` fields don't already provide —
and a field is cheap to change where a directory tree is not. A page that turns out to be
two topics gets recategorized with one edit. Do not create `concepts/`, `entities/`, or
`analyses/` subfolders. `sources/` is separate only because those pages are 1:1 with
`raw/`.

There is no `log.md`; git is the log (see [Commits](#commits)). `index.md` exists but is
**not** a page catalog — that is derived (see [Finding pages](#finding-pages)).

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

`type` is `page`, `source`, or `index` (only `wiki/index.md` is the last).

`summary` is the retrieval hook — it is what a future session reads to decide whether to
open this page, so write it to answer *what questions does this page settle?* rather than
to describe the page. One line, always double-quoted (summaries tend to contain colons).

It overlaps with the page's opening lede, and that is deliberate — the two have different
readers. `summary` is a ~15-word key an agent greps to decide whether to open the file at
all; the lede is prose that orients a human who already opened it, and it is what Obsidian
shows on hover-preview and in search results. Writing one to serve both jobs makes it bad
at both. What they must not do is *disagree*: if the lede has moved on and the summary
hasn't, the catalog is quietly lying. Lint checks for that.

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
- **A lede paragraph directly under the title**, before any `##` section. It must stand
  alone: someone who reads only the lede should come away with the point of the page.
  No page opens straight into a section heading.
- Filenames are lowercase and hyphenated: `tool-use.md`, `context-window.md`.
- Link generously with `[[wikilinks]]`. A link to a page that doesn't exist yet is a
  legitimate marker of a gap — it shows up in Obsidian's Unresolved links pane.
- Concept pages state claims and cite the source page the claim came from:
  `... one page revised by many sources ([[karpathy-llm-wiki-gist]]).`
- When two sources disagree, say so on the page. Don't silently pick a winner.

## Finding pages

Two different things, and keeping them separate is the whole design:

- **`wiki/index.md` is the map.** One row per topic area: what belongs in it, where to
  start, what it still lacks. It answers *where does a new page go* and *what don't we
  know yet*. It does not list pages, so it does not go stale as pages change.
- **The catalog is derived.** Page-level detail — every page with its category and
  summary — comes from a command, so it is always current.

Read `index.md` when you need orientation or are deciding where a new page belongs. Run
the catalog when you need to know what actually exists:

```bash
head -n 12 wiki/*.md wiki/sources/*.md | grep -E '^(==>|category:|summary:)'
```

That is the first command of most sessions. It prints every page with its category and
one-line summary, always current, because there is no second copy to fall behind.

The division of labour is: **a property of one page lives in that page's frontmatter; a
statement about the collection lives in `index.md`.** Never copy a page's summary into
`index.md` — that is the drift that got the old catalog deleted.

Narrow it when you already know the shape of the question:

```bash
grep -rl  --include='*.md' '^category: tool-use-protocols' wiki/  # one category
grep -rlE --include='*.md' '^tags:.*[][, ]memory[],]' wiki/       # one tag
grep -rli --include='*.md' 'speculative decoding' wiki/           # full text
grep -rl  --include='*.md' '\[\[mcp\]\]' wiki/                     # what links here
```

`category:` is a single value, so it matches exactly. `tags:` is an inline array, so a
tag match has to anchor on the surrounding `[`, `,`, `]` or space. **Do not reach for
`grep -w`** — a hyphen is a word boundary, so `grep -w memory` also matches the page
tagged `context-memory`. That is the one place a naive pattern silently returns wrong
pages rather than no pages.

For browsing tags by hand, Obsidian's tag pane is better than any of this: it is exact,
already enabled, and it lists counts.

Use the derived catalog to *choose* pages and grep to *catch what it missed*. A summary
is one line and will not mention everything a page covers, so when a question doesn't map
cleanly onto a summary, grep the full text before concluding the wiki has no answer.

There is deliberately no saved *view* file (no `.base`). A saved query would only serve
the human anyway — reading a `.base` gives you the query, not the results. For browsing
inside Obsidian, use `index.md`, the tag pane, the graph, and search.

## Workflows

### Ingest

One raw item at a time. The cost is paid once here so that later queries read two pages
instead of re-reading the source.

1. Read `wiki/index.md` for the map, then run the catalog command (see
   [Finding pages](#finding-pages)) to see what actually exists.
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
   they fit. That alone puts it in the catalog — **do not add a row to `index.md`.**
7. Touch `index.md` only if this source changed the *map*: a new topic area, an area
   that now has a first page worth naming as `Start here`, or a `Wanted` gap this
   source just filled. Most ingests leave it alone.
8. Commit.

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

# pages that open straight into a section, with no lede
for f in wiki/*.md wiki/sources/*.md; do
  awk 'NR==1 && /^---$/{infm=1; next} infm && /^---$/{infm=0; done=1; next} infm{next}
       done && /^# /{t=1; next}
       t && NF { if ($0 ~ /^#/) print FILENAME": no lede"; exit }' "$f"
done

# categories used by a page but missing from the index map, and vice versa
# ('meta' is index.md's own category, not a topic area)
comm -3 <(head -n 12 wiki/*.md wiki/sources/*.md | sed -n 's/^category: //p' \
            | grep -v '^meta$' | sort -u) \
        <(sed -n 's/^| `\([a-z-]*\)`.*/\1/p' wiki/index.md | sort -u)
# left column  = used on a page but unmapped -> add it to index.md
# right column = mapped with no pages yet    -> that is a declared gap, fine

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
- summaries that no longer match what the page grew into, or contradict its own lede
- pages that open straight into a `##` section with no lede
- `index.md` rows whose `Start here` or `Wanted` no longer reflect what the wiki holds

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
- `index.md` has one row per topic area, never one per page. If you are adding a row
  because you added a page, stop: the catalog already covers it.
- New raw files are named `YYYY-MM-DD-<slug>.<ext>`.
