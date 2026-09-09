---
type: source
tags: [knowledge-base, memory, retrieval, agent-workflow]
raw: raw/2026-09-09-karpathy-llm-wiki-gist.md
url: https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f
---

# Karpathy — LLM Wiki

Andrej Karpathy's gist describing the [[llm-wiki-pattern]]: instead of retrieving from
raw documents at query time, have an LLM agent incrementally build and maintain a
persistent, interlinked markdown wiki that sits between you and the sources. The
document is deliberately abstract — it states the pattern and expects you to instantiate
it with your agent. This vault is one instantiation.

## The argument against query-time retrieval

RAG rediscovers knowledge from scratch on every question. Nothing accumulates. A
question that needs five documents synthesized forces the model to find and reassemble
the fragments every single time.

The wiki inverts the cost: the synthesis is done once, at ingest, and then *kept
current*. Cross-references are already there, contradictions already flagged, and the
summary already reflects everything read so far. Karpathy calls the wiki a "persistent,
compounding artifact" — the value comes from that compounding, not from any single page.

## Division of labor

The human never (or rarely) writes the wiki. The human curates sources, explores, and
asks the right questions; the agent does the summarizing, cross-referencing, filing, and
bookkeeping. Karpathy's own setup runs the agent on one side and Obsidian on the other,
browsing the results live: "Obsidian is the IDE; the LLM is the programmer; the wiki is
the codebase."

## Three layers

- **Raw sources** — curated source documents, immutable. The agent reads but never
  modifies them. Source of truth.
- **The wiki** — agent-generated markdown: summaries, entity pages, concept pages,
  comparisons, syntheses. The agent owns this layer entirely.
- **The schema** — `CLAUDE.md` / `AGENTS.md`, telling the agent the structure,
  conventions, and workflows. Named as "the key configuration file — it's what makes the
  LLM a disciplined wiki maintainer rather than a generic chatbot," and expected to
  co-evolve with the user over time.

## Three operations

**Ingest** — drop a source in, agent reads it, discusses takeaways with you, writes a
summary page, updates the index, and revises the entity and concept pages it touches.
Karpathy notes a single source might touch 10–15 wiki pages, and prefers ingesting one
at a time while staying involved.

**Query** — agent finds relevant pages, reads them, answers with citations. The insight
he flags as important: **good answers get filed back into the wiki as new pages**, so
explorations compound the same way ingested sources do rather than vanishing into chat
history.

**Lint** — periodic health check for contradictions, stale claims superseded by newer
sources, orphan pages, concepts mentioned but lacking a page, missing cross-references,
and gaps a web search could fill.

## index.md and log.md

Two navigation files, with different jobs. `index.md` is content-oriented: a catalog of
every page with a link and a one-line summary, organized by category, updated on every
ingest, and read first when answering a query. He claims this "works surprisingly well
at moderate scale (~100 sources, ~hundreds of pages) and avoids the need for
embedding-based RAG infrastructure." `log.md` is chronological and append-only — a
record of ingests, queries, and lint passes, greppable if entries use a consistent
prefix.

*(This vault keeps `index.md` and drops `log.md`; see [[llm-wiki-pattern]] for why.)*

## Optional pieces

Search tooling once the index stops scaling — [qmd](https://github.com/tobi/qmd) (local
hybrid BM25 + vector + LLM reranking, CLI and MCP server) is suggested. Obsidian Web
Clipper for getting articles in as markdown; a hotkey to download attachments locally so
the agent can view images rather than depend on live URLs; Obsidian's graph view for
seeing hubs and orphans; Marp for slides; Dataview for querying frontmatter.

## Why it works

The bottleneck in a knowledge base is bookkeeping, not reading or thinking. Humans
abandon wikis because maintenance cost grows faster than value. "LLMs don't get bored,
don't forget to update a cross-reference, and can touch 15 files in one pass. The wiki
stays maintained because the cost of maintenance is near zero."

Karpathy places the idea in the lineage of Vannevar Bush's Memex (1945) — a private,
curated store with associative trails between documents, where the connections matter as
much as the documents. The part Bush couldn't solve was who does the maintenance.
