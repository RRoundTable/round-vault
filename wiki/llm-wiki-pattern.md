---
type: page
tags: [knowledge-base, memory, retrieval, agent-workflow]
category: context-memory
summary: "Compile sources into a maintained wiki at ingest rather than retrieving from them at query time, so knowledge compounds on shared pages."
---

# LLM wiki pattern

A memory architecture for LLM agents: rather than retrieving from raw documents at query
time, an agent **compiles** sources into a persistent, interlinked set of markdown pages
and maintains them as new sources arrive. Queries then read the compiled layer, not the
sources ([[karpathy-llm-wiki-gist]]).

This vault is an instance of the pattern. `CLAUDE.md` is its schema.

## The move: compile-time instead of query-time

RAG does its synthesis at query time and throws it away. Ask a question needing five
documents, and the model finds and reassembles the fragments; ask a related question
tomorrow and it does the whole thing again. Nothing accumulates.

The wiki moves that work to ingest. A source is read **once**, and the understanding it
produced is written down where the next query can find it already assembled. The
economics are the point: ingest is expensive and happens once per source; queries are
cheap and happen constantly.

The consequence is that the wiki *compounds*. A page like `tool-use.md` is revised by
every source that touches tool use, so it ends up holding a synthesis no single source
contains. That is the payload — not any individual page, but what many sources deposit
on the same page over time.

## Three layers, and why the boundary matters

| Layer | Owner | Mutability |
| --- | --- | --- |
| `raw/` | the human curates | immutable |
| `wiki/` | the agent | rewritten constantly |
| schema (`CLAUDE.md`) | co-evolved | changes when the workflow does |

The immutability of `raw/` is load-bearing, not hygiene. It keeps ground truth from
blurring with interpretation, which is what makes the wiki **recompilable** (throw the
wiki away and rebuild it), **verifiable** (every claim traces to an untouched artifact),
and **auditable** (a wrong claim is the agent's error, never a corrupted source). An
agent that "fixes" a source destroys all three at once.

The schema is the layer people skip and shouldn't. Without it each session reinvents the
structure and the wiki drifts into inconsistency — which is exactly the maintenance
failure the pattern is supposed to solve.

## The catalog should be derived, not stored

The gist's retrieval layer is `index.md`: every page listed with a one-line summary,
grouped by category, updated on every ingest. The agent reads it first and opens only
what it points at. The claim is that at the scale a personal wiki reaches (~100 sources),
an LLM reading a table of contents beats embedding search and carries none of the
infrastructure ([[karpathy-llm-wiki-gist]]).

The retrieval argument is right. Storing it in a separate file is not.

A hand-written index is a **denormalized cache with no invalidation**. The one-line
summary duplicates knowledge that belongs to the page, and the copy can drift from the
original with nothing forcing anyone to notice. The usual patch is a discipline rule —
*always update the index in the same commit* — which is a rule that exists solely to
protect the cache. That's the tell.

The same information normalizes cleanly onto the page it describes:

```yaml
category: context-memory
summary: "One line saying what this page is about."
```

and the catalog becomes a projection over the pages rather than a file:

```bash
head -n 12 wiki/*.md wiki/sources/*.md | grep -E '^(==>|category:|summary:)'
```

That output is what `index.md` contained, and it cannot be stale, because there is no
second copy to fall out of date. The invariant disappears rather than being enforced.

What survives from the original argument is the part that was never really about the
file. Summaries earn their place because **filenames don't disambiguate** — `ls` says
`context-window.md` exists, not whether it covers token limits, attention cost, or
compaction — and because **grep matches strings, not concepts**: a question about how
agents remember things across sessions won't hit a page that never uses the word
"remember." Letting a model read a compact list of summaries *is* semantic retrieval;
it just ships as text in the prompt instead of as a vector store.

What doesn't survive is the *catalog* function of the file. Three things do, and they
are exactly what a projection cannot express:

- **Absence.** A derived catalog lists categories that have pages. It cannot show
  "Evaluation — nothing here yet", which is a fact about the topic, not about any page.
- **Curation.** Where to start, which page is central, what order to read in. A
  projection weights every page equally because it has nothing to weight them by.
- **Intent.** What this area still needs, in sources or open questions. Nothing in the
  existing pages can imply what is missing from them.

So `index.md` comes back, but as a **map, not a catalog**: one row per topic area, never
one per page. That inversion is what matters. A per-page row duplicates the page and
drifts; a per-area row describes the shape of the subject, which changes when your
understanding changes rather than when a file does. The rule that falls out of it:
a property of one page lives in that page's frontmatter, and a statement about the
collection lives in the index.

## Two summaries, on purpose

Every page states its point twice: once as `summary:` frontmatter, once as the lede
paragraph under the title. That looks like the duplication this design otherwise refuses,
and it isn't, because the two have different readers and different lengths.

`summary` is a ~15-word retrieval key. An agent greps it to decide whether to open the
file *at all*, so it is written to answer *what questions does this page settle?* The
lede is prose for someone who already opened the page — and it is what Obsidian shows on
hover-preview and in search results, which is how a human browses without any index. One
string cannot do both jobs: tuned for retrieval it reads as a stub, tuned for reading it
is too long to scan forty of.

The distinction that matters is **same function versus same subject**. `index.md`'s old
per-page rows and the pages themselves performed the *same function* — cataloguing — in
two places, so one was redundant and drifted. A summary and a lede perform *different
functions* on the same subject. Collapsing those degrades both; the correct control is
not deduplication but a consistency check, so lint verifies they still agree rather than
demanding there be only one.

The general rule: duplication is a problem when two copies answer the same question, not
when two artifacts serve different readers.

## What ingest actually costs

The fan-out is the whole mechanism. A source landing in `raw/` produces:

```
raw/2026-09-09-mcp-spec.md
  -> derive catalog           what already exists?
  -> wiki/sources/mcp-spec.md new source page   (category + summary)
  -> wiki/mcp.md              new concept page  (category + summary)
  -> wiki/tool-use.md         revised: now links [[mcp]]
  -> commit
```

One source, four pages touched. A source that produces only its own source page means
the integration step was skipped, and the wiki has become a pile of summaries rather
than a knowledge base. Karpathy puts the expected fan-out at 10–15 pages on a mature
wiki.

## Queries feed back in

Answers worth keeping — a comparison, a synthesis, a connection nobody had written down
— get filed as new wiki pages. Explorations compound like ingested sources instead of
disappearing into chat history. This is what makes the wiki grow from use, not just from
reading.

## Deviations in this vault

Four, each trading a piece of the gist for something cheaper:

**Flat `wiki/`, no `concepts/` + `entities/` + `analyses/`.** That split is a taxonomy
chosen before there is content to taxonomize, and Obsidian resolves `[[mcp]]` regardless
of folder — so the folders buy nothing a `category:` field doesn't. Grouping lives where
it's cheap to change: recategorizing a page is one edit, moving it is a rename plus every
reference to it. `sources/` stays separate because those pages
are 1:1 with `raw/` and serve as the citation targets.

**No `log.md`.** Git covers it, and better: a commit records what changed, while a log
entry records what the agent *claims* changed. The two diverge exactly when it matters.
`git log --grep '^ingest:'` reconstructs the timeline; `git log -S'<claim>'` finds when a
claim entered — which no append-only log can answer at all.

**`index.md` is a map, not a catalog.** Page-level detail is derived from `category:`
and `summary:` frontmatter by one shell command, so it can't drift. The index holds only
what a projection can't express — topic areas including empty ones, entry points, and
known gaps — so it stays one row per area and doesn't grow with the wiki. There is no
saved view file (`.base`) either: a stored query serves only the human, and reading one
gives an agent the query rather than the results. See above.

**No lint script.** Obsidian's Unresolved links pane already covers broken references,
and `ls raw/` against `ls wiki/sources/` covers un-ingested sources. Everything else
worth checking — contradictions, superseded claims, concepts that deserve a page, pages
that should be split — is semantic. Lint is an LLM workflow with a few shell commands in
front of it, not a script.

## Convergent designs

The same discipline shows up in agent memory research that has nothing to do with wikis.
**ACE** treats an agent's context as an evolving playbook of `(identifier, description)`
bullets and forbids its curator from rewriting the whole blob, because iterative rewrites
produce *context collapse* (detail shed until a specific playbook has become a generic
one) and *brevity bias* (each rewrite looks like an improvement because shorter reads
better, and the ratchet turns one way only). Its fix is incremental itemized updates merged
by deterministic logic ([[agentic-context-engineering]]).

That is this vault's reason for immutable `raw/` and for lint *proposing* fixes rather than
rewriting pages, arrived at from the opposite end. The shared claim: **additions and targeted
revisions compound; wholesale rewrites erode.**

It is specifically *not* the argument for deriving the catalog. That one is about a
denormalized cache with no invalidation — two copies drifting apart — which is a different
failure from detail lost under compression. The two get conflated easily because both end in
"don't keep a second, staler version," but the mechanisms and the fixes differ: deduplication
cures drift, and only refusing to summarize cures collapse.

Where the two differ is what the entries hold — ACE's bullets are procedural insights
mined from the agent's own rollouts and consumed by that same agent, while wiki pages are
declarative knowledge compiled from external sources for future sessions and a human
reader. That two designs with different content, different producers and different readers
land on the same maintenance rule is mild evidence the rule is real rather than an artifact
of either one.

MCE goes one step further and separates the *mechanism* from the *artifact* — optimizing
the context-management skill at a meta level and the context itself at a base level. The
split maps exactly onto `CLAUDE.md` versus `wiki/`, which is why they are separate files
that change on different clocks.

## Lineage

Vannevar Bush's Memex (1945): a private, curated document store with associative trails,
where the connections between documents matter as much as the documents. Bush's design
never said who maintains the trails. Humans abandon wikis because maintenance cost grows
faster than value; an agent that doesn't get bored and can revise fifteen files in one
pass drives that cost near zero, which is the only thing that changed.
