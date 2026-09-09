---
type: page
tags: [knowledge-base, memory, retrieval, agent-workflow]
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

## index.md is the retrieval layer

The catalog — every page, one line each, grouped by category — is what this pattern uses
*instead of* a vector database. The agent reads it first, decides which two or three
pages to open, and opens only those. At the scale a personal wiki actually reaches
(~100 sources), an LLM reading a table of contents beats embedding search, and it
carries none of the infrastructure.

It only works if it's maintained, which is why updating it is a step in the ingest
workflow rather than a periodic cleanup.

## What ingest actually costs

The fan-out is the whole mechanism. A source landing in `raw/` produces:

```
raw/2026-09-09-mcp-spec.md
  -> read index.md            what already exists?
  -> wiki/sources/mcp-spec.md new source page
  -> wiki/mcp.md              new concept page
  -> wiki/tool-use.md         revised: now links [[mcp]]
  -> wiki/index.md            new row
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

Three, each trading a piece of the gist for something cheaper:

**Flat `wiki/`, no `concepts/` + `entities/` + `analyses/`.** That split is a taxonomy
chosen before there is content to taxonomize, and Obsidian resolves `[[mcp]]` regardless
of folder — so the folders buy nothing that `index.md` and a `type:` field don't.
Grouping lives where it's cheap to change. `sources/` stays separate because those pages
are 1:1 with `raw/` and serve as the citation targets.

**No `log.md`.** Git covers it, and better: a commit records what changed, while a log
entry records what the agent *claims* changed. The two diverge exactly when it matters.
`git log --grep '^ingest:'` reconstructs the timeline; `git log -S'<claim>'` finds when a
claim entered — which no append-only log can answer at all.

**No lint script.** Obsidian's Unresolved links pane already covers broken references,
and `ls raw/` against `ls wiki/sources/` covers un-ingested sources. Everything else
worth checking — contradictions, superseded claims, concepts that deserve a page, pages
that should be split — is semantic. Lint is an LLM workflow with a few shell commands in
front of it, not a script.

## Lineage

Vannevar Bush's Memex (1945): a private, curated document store with associative trails,
where the connections between documents matter as much as the documents. Bush's design
never said who maintains the trails. Humans abandon wikis because maintenance cost grows
faster than value; an agent that doesn't get bored and can revise fifteen files in one
pass drives that cost near zero, which is the only thing that changed.
