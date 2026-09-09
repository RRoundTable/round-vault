# round-vault

An LLM-maintained wiki on **LLM-agent-native systems**, built on the
[LLM wiki pattern](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f):
an agent compiles knowledge out of raw sources into an interlinked set of markdown
pages, and keeps them current. You curate sources and ask questions; the agent writes.

## Structure

| Path | Purpose |
| --- | --- |
| `raw/` | Sources, verbatim and immutable. Read once, at ingest. Never edited. |
| `wiki/index.md` | The map: topic areas, entry points, known gaps. One row per area. |
| `wiki/sources/` | One ~1-page summary per raw item. The citation targets. |
| `wiki/*.md` | Concept and entity pages, flat. Where knowledge compounds. |
| `assets/` | Images. |
| `CLAUDE.md` | The schema: page format, the ingest/query/lint workflows, invariants. |

`CLAUDE.md` is the important file. Without it every session reinvents the structure and
the wiki drifts. Point an agent at this repo and it reads that first.

Using Cursor or Codex instead? `ln -s CLAUDE.md AGENTS.md`.

## Working in it

Open this directory — the repo root — as an Obsidian vault. Ask an agent to ingest,
query, or lint; the workflows are in `CLAUDE.md`.

`wiki/index.md` is the map — one row per topic area, with what belongs in it, where to
start, and what it still needs. It never lists individual pages, so it doesn't go stale.

Page-level detail is derived instead, from each page's own `category:` and `summary:`:

```bash
head -n 12 wiki/*.md wiki/sources/*.md | grep -E '^(==>|category:|summary:)'
```

The split: a property of one page lives in that page's frontmatter; a statement about
the collection lives in the index.

There is no changelog file either. Git is the log:

```bash
git log --oneline --grep '^ingest:' | head    # recent activity
git show --stat <sha>                         # what one ingest touched
git log -S'<claim text>' -- wiki/             # when a claim entered the wiki
```
