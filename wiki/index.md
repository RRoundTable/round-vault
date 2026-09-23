---
type: index
category: meta
summary: "Map of the wiki's topic areas: what belongs in each, where to start, and what each still needs."
---

# Index

The map of this wiki — one row per topic area, not per page. It answers *where does a new
page go*, *where do I start reading*, and *what is this area still missing*: the three
things that cannot be derived from the pages themselves.

Page-level detail is **not** listed here, because it would go stale. Derive it instead:

```bash
head -n 12 wiki/*.md wiki/sources/*.md | grep -E '^(==>|category:|summary:)'
```

## Topic areas

`Category` is the value that goes in a page's `category:` frontmatter. `Covers` is the
rule for choosing it. An area with no `Start here` has no pages yet — that is a gap, and
it is the point of listing it.

| Category              | Covers                                                                                                          | Start here                   | Wanted                                                                                                                               |
| --------------------- | --------------------------------------------------------------------------------------------------------------- | ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| `agent-architectures` | Decomposition of agent systems: planning, delegation, multi-agent structure, control loops.                     | [[harness]]                  | A real multi-agent postmortem; something on when delegation stops paying; full reads of AHE and Hyperagents (only abstracts so far). |
| `context-memory`      | What the agent knows and where it keeps it: context windows, compaction, persistent stores, retrieval strategy. | [[agentic-context-engineering]] | Compaction strategies under long sessions; how memory is evicted.                                                                    |
| `tool-use-protocols`  | How agents reach the outside world: tool definitions, MCP, function calling, execution interfaces.              | —                            | The MCP spec itself; something on tool-definition design that isn't vendor marketing.                                                |
| `evaluation`          | Measuring agent systems: benchmarks, LLM-as-judge, trajectory scoring, regression testing.                      | [[auto-research-evaluation]] | A trajectory-scoring method with a real failure analysis; a verifier design that survives an adversarial agent.                      |
| `runtime-serving`     | What runs the agent: inference serving, caching, latency and cost, scheduling, long sessions.                   | —                            | Prefix-cache behaviour under agent workloads.                                                                                        |
| `machine-native-models` | Models built for software, not people, as the consumer: typed outputs, calibrated probabilities, and the training objectives behind them. | [[jev]]                      | Any independent calibration measurement (reliability curve, ECE) of a decision model; a primary account of RLCD; third-party benchmarks of Jev.                            |
| `safety-sandboxing`   | Permissioning, isolation, prompt injection, containment of side effects.                                        | [[harness-reward-hacking]]   | A concrete prompt-injection case study against a tool-using agent; how editable-surface whitelists are enforced in practice.         |

Sources are a `type`, not a topic area — each one also carries the `category` of what it
is about. List them with `ls wiki/sources/`.
