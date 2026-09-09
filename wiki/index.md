---
type: page
tags: [index]
---

# Index

Catalog of every page in this wiki. Read this first — it is the retrieval layer. Find
the pages a question touches here, then open only those.

Categories are a convenience, not a filesystem: every page lives flat in `wiki/`.

## Agent architectures

How agent systems are decomposed — planning, delegation, multi-agent structure, control
loops.

*(no pages yet)*

## Context & memory

What the agent knows and where it keeps it: context windows, compaction, persistent
stores, retrieval strategy.

| Page | Summary |
| --- | --- |
| [[llm-wiki-pattern]] | Compile sources into a maintained wiki at ingest instead of retrieving from them at query time; knowledge compounds on shared pages. |

## Tool use & protocols

How agents reach the outside world — tool definitions, MCP, function calling, sandboxed
execution interfaces.

*(no pages yet)*

## Evaluation

Measuring agent systems: benchmarks, LLM-as-judge, trajectory scoring, regression
testing.

*(no pages yet)*

## Runtime & serving

What runs the agent — inference serving, caching, latency and cost, scheduling,
long-running sessions.

*(no pages yet)*

## Safety & sandboxing

Permissioning, isolation, prompt injection, containment of agent side effects.

*(no pages yet)*

## Sources

One page per item in `raw/`. These are the citation targets.

| Source | Summary |
| --- | --- |
| [[karpathy-llm-wiki-gist]] | Karpathy's gist proposing an agent-maintained wiki over query-time RAG: three layers (raw/wiki/schema), three operations (ingest/query/lint). |
