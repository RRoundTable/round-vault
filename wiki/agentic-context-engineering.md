---
type: page
tags: [context-engineering, memory, knowledge-base, self-improvement, harness]
category: context-memory
summary: "Treat context as an evolving playbook of itemized entries rather than a prompt to rewrite — and then optimize the mechanism that maintains it, not just its contents."
---

# Agentic context engineering

Appending every tool response and model generation to the context stops working as soon as
the job horizon gets long. **Agentic Context Engineering** (ACE; Zhang et al. 2025) is the
answer that treats context as an *evolving playbook* rather than an ever-lengthening
prompt. Its central design choice is a constraint on how the playbook may be updated —
never rewrite the blob, emit itemized entries — because the failure it is defending against
comes from rewriting, not from length ([[lilian-weng-harness-engineering]]).

Context engineering sits on the second rung of the harness optimization ladder, between
prompts and workflow; see [[self-improving-harness]] for the rest of the climb.

## ACE: three roles over one playbook

The playbook is a set of bullet points, each with an identifier and a description.
Three components maintain it:

- **Generator** — produces task trajectories, referring to the bullets.
- **Reflector** — distills insights from trajectories, *successful and failed alike*.
- **Curator** — updates the structured context with incremental, itemized entries.

The load-bearing constraint is on the curator: it **does not rewrite a full prompt blob**.
It emits structured bullets as `(identifier, description)`, which are merged into the
playbook by deterministic logic, and periodically refined and deduplicated.

The two failure modes this defends against are worth naming, because they are what an
LLM-maintained store degrades into if you let a model rewrite it wholesale:

- **Context collapse** — successive rewrites shed detail until a long, specific playbook
  has been compressed into a short, generic one.
- **Brevity bias** — each individual rewrite looks like an improvement, because shorter
  reads better, and the ratchet only turns one way.

The structural fix is the same one behind not letting an agent regenerate `index.md` from
scratch each session: **make the update incremental and itemized, so that the merge is
mechanical and no pass ever has license to restate the whole thing.** An entry survives
unless something specifically removes it. This vault takes the identical position on
`raw/` immutability and on deriving the catalog instead of regenerating it — additions and
targeted revisions compound, wholesale rewrites erode.

ACE reports +10.6% on agents and +8.6% on finance with reduced adaptation latency and
rollout cost, and two results the survey omits ([[harness-survey-arxiv-abstracts]]). It
adapts **without labeled supervision**, using natural execution feedback alone — which is
what makes it deployable on tasks that have no training set. And on the AppWorld
leaderboard it matches the top production-level agent on the overall average while
*surpassing* it on the harder test-challenge split, using a smaller open-source model. A
better-maintained context substituting for a bigger model is the cleanest evidence available
that this layer is worth engineering.

That ACE learns its entries from rollouts is what moves it toward self-managed memory. But
its update rules and workflow are still handcrafted, which is the opening for the next
step.

## MCE: optimize the mechanism, not the artifact

**Meta Context Engineering** (MCE; Ye et al. 2026) separates *how to manage context* from
*what is in context*, and optimizes both at different levels.

A skill $s$ defines a context function $c_s = (\rho_s, F_s)$ mapping input $x$ to context
$c = F_s(x; \rho_s)$, where $\rho_s$ are **static** components (prompts, knowledge bases,
code libraries) and $F_s$ are **dynamic** operators (search, selection, filtering,
formatting). The optimization is bi-level:

$$
\text{Inner: } c_s^*=\arg\max_{c_s}J_\text{train}(c_s;s) \quad
\text{Outer: } s^*=\arg\max_{s\in\mathcal{S}}J_\text{val}(c_s^*)
$$

The inner loop finds the best context given a skill on training data; the outer loop finds
the skill that generalizes best on validation. A skill database keeps the history
$\mathcal{H}_{k-1} = \{(s_i, c_i, J_i^\text{train}, J_i^\text{val})\}$, and a meta-level
agent performs agentic **crossover** over prior skills to propose the next one,
$s_k = \text{crossover}(\tau, \mathcal{H}_{k-1})$. A base-level context engineer then
executes it and learns the context function from rollout feedback,
$c_k = \text{engineer}(\tau, s_k; c_{k-1}^*, \mathcal{R}_k)$.

MCE is explicitly a critique of ACE's shape, and says so: prior CE methods rely on
"rigid generation-reflection workflows and predefined context schemas" that "impose
structural biases and restrict context optimization to a narrow, intuition-bound design
space" ([[harness-survey-arxiv-abstracts]]). It reports 5.6–53.8% relative improvement over
prior agentic CE methods across five domains, mean 16.9%.

The disagreement is real and worth not smoothing over. ACE's fixed itemized schema is what
*prevents* collapse — determinism in the merge is the whole defense. MCE gives that up for a
larger design space and relies on the outer validation loop to catch degradation instead.
Whether free-form skills reintroduce brevity bias by another route is not something either
abstract settles.

Where ACE fixes a heuristic structure for context, MCE uses **free-form skills** and lets
the structure evolve. The implementation is the part worth stealing: a context function is
just **a directory of files** — `skill.md` for the static part, context and data rollouts
for the dynamic part — and both optimization levels run in an ordinary agentic coding
environment with the standard tool set
$\mathcal{T}=\{\texttt{Read},\texttt{Write},\texttt{Edit},\texttt{Bash},\texttt{Glob},\texttt{Grep},\texttt{TodoWrite}\}$.
No special optimizer machinery; the filesystem is the data structure and a coding agent is
the optimizer. Same bet as [[harness]]'s file-system-as-memory pattern.

## Over-compression is the recurring enemy

Two systems on different rungs of the ladder name the same failure, which is some evidence
it is structural rather than incidental:

| System | Name for it | Where the loss happens |
| --- | --- | --- |
| ACE | brevity bias / context collapse | iterative rewrites of the stored artifact |
| Meta-Harness | "text optimizers compress feedback too aggressively" | a summarizer between the evidence and the proposer |

These are not quite the same mechanism. ACE's loss is cumulative — each rewrite looks like an
improvement and the ratchet turns one way. Meta-Harness's is single-step: the proposer never
sees the detail at all, because something summarized the trace before handing it over. Its
fix is correspondingly different — not a better merge rule but **filesystem access to the raw
traces, source and scores, so the agent can `grep` for what it needs**
([[harness-survey-arxiv-abstracts]]).

What they share is the underlying reason summarization is dangerous here: it is lossy in a
direction you cannot predict at summarization time, because which detail matters is determined
by the *next* decision, not the current one. That makes a summary sound as an index and unsound
as a replacement. AHE's layered drill-down — per-task reports over raw traces, with the raw
traces still reachable — is the resolution the harness literature converges on
([[self-improving-harness]]).

## The mechanism/artifact split generalizes

MCE's separation is the same move this vault makes with its schema. `CLAUDE.md` is the
mechanism — how pages are structured, when to ingest, what makes a page findable — and
`wiki/` is the artifact. They evolve on different clocks and for different reasons: a page
changes when a source arrives, the schema changes when the workflow proves wrong. Merging
them means every ingest can silently rewrite the rules, which is exactly the collapse
dynamic ACE guards against, one level up.

Weng lists the context and memory lifecycle as an open bottleneck and takes the strong
position that context engineering "will and should become a core part of intelligence,
rather than staying in the software system layer" — with the human analogy that we
maintain memory across a lifetime. The counter-consideration on
[[harness]] applies: prompt engineering was largely internalized by instruction tuning,
but the *need to specify* goals, constraints and context never went away. Something
similar likely holds here — the maintenance heuristics get absorbed, the store does not.

## Where this disagrees with query-time retrieval

ACE and the [[llm-wiki-pattern]] both refuse the same thing: keeping sources whole and
re-synthesizing at query time. Both pay at write time to make read time cheap, and both
insist the written artifact be *edited incrementally* rather than regenerated.

They differ in what the entries are for. ACE's playbook bullets are procedural — insights
about how to do a task, mined from the agent's own rollouts, consumed by the same agent.
Wiki pages are declarative — knowledge compiled from external sources, consumed by future
sessions and by a human. The maintenance discipline is nonetheless identical, which is
mild evidence it is the right discipline rather than a coincidence of either design.
