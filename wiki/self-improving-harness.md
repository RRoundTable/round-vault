---
type: page
tags: [harness, self-improvement, evolutionary-search, agent-workflow, code-generation]
category: agent-architectures
summary: "Once a harness is code, an agent can search it — the ladder from prompts to optimizer code, and why the base model's capability gates the whole loop."
---

# Self-improving harness

A [[harness]] is code: it programs how prompts, tool calls, subagents, control flow,
memory and workflow logic fit together. That single observation is what turns harness
design into an optimization problem, because **an LLM that can edit code can edit its own
harness**, and the design space reachable that way is far larger than the one reachable
by rewriting prompts ([[lilian-weng-harness-engineering]]). This page covers what happens
when you actually run that loop, and the two results showing why it doesn't work for
everyone.

## The ladder

The object being optimized has climbed steadily:

```
instruction prompts → structured context → workflow → harness code → optimizer code
```

Each rung replaces a hand-written artifact with a searchable one, and each becomes viable
only when the base model is strong enough to search it. The rungs are not alternatives —
a real system optimizes several at once, which is the argument for treating the whole
harness as one object rather than tuning context and workflow separately.

Rung two is [[agentic-context-engineering]]. Rung three, workflow, went the same way:
**ADAS** (Hu et al. 2025) formulates agent design as "meta-agent search" — keep an archive
of agentic workflows seeded with CoT and self-refine, have a meta-agent describe then
*program* a new one inspired by the archive, run two self-refine passes to check novelty,
evaluate, and add the survivors back. **AFlow** (Zhang et al. 2025) instead makes the
workflow a graph — nodes are LLM-invoking actions, edges are logical operations in code —
and runs MCTS over it, expanding a selected node by asking an LLM for a modification
conditioned on that node's measured performance, stopping when the top-$k$ average score
plateaus. AFlow beat both manual workflows and ADAS on QA, code and math.

Rung four is where the harness itself becomes the artifact, and rung five is
**Meta-Harness** (Lee et al. 2026), which optimizes *the code that decides what to store,
retrieve and present* — a harness for optimizing harnesses. Two implementation details
recur across every system at this level and are worth stating as the pattern:

- The execution history lives in the file system, and the proposing agent reads it with
  `grep` and `cat` rather than having it shoveled into one prompt.
- A candidate harness is a **directory**: its own source code, its scores, its rollout
  trajectories, its state updates. Candidates are kept as a Pareto frontier, not a single
  best.

## STOP, and the capability floor

**Self-Taught Optimizer** (Zelikman et al. 2023) is the early statement of the recursion.
A seed improver takes a solution, a utility function and a black-box model and returns a
better solution, $s' = I(u, s; M)$. The goal is not to improve $s$ but to improve $I$.
Define meta-utility as the improver's average utility across downstream tasks,

$$
\hat{u}(I) \triangleq \frac{1}{\vert\mathcal{D}\vert}\mathbb{E}_{(u,s)\sim \mathcal{D}}[u(I(u,s; M))]
$$

and then apply the improver to itself: $I_t = I_{t-1}(\hat{u}, I_{t-1}; M)$. The improved
improvers discovered genetic algorithms, decomposition, multi-armed prompt bandits,
simulated annealing, temperature variation, and beam/tree search — i.e. they rediscovered
the optimizer literature, which is the encouraging part.

The discouraging part is the headline result of this whole area: **STOP improved mean
downstream performance with GPT-4 and degraded it with GPT-3.5 and Mixtral.** Recursive
structure alone buys nothing. There is a capability floor below which the loop is
actively harmful, because a weak model's edits are worse than no edits and the recursion
compounds them.

## Updating is easy; benefiting is not

Lin et al. (2026) sharpen this by splitting the capability in two:

| Axis | What it measures | How it scales |
| --- | --- | --- |
| harness-**updating** | producing a useful harness edit | roughly **flat** across model sizes |
| harness-**benefit** | exploiting an updated harness | **non-monotonic**, peaks mid-tier |

From Qwen3.5-9B to Claude Opus 4.6, updating capability barely moves — the 9B proposer
writes a skill *procedurally isomorphic* to the one Opus writes. What separates models is
the ability to use the result: invoking skills and tools correctly and at the right
moment, and following instructions over a long horizon.

This inverts the intuitive division of labor. The scarce resource is not the intelligence
that authors the harness but the discipline that executes under it — which is why a small
model can be the proposer, and why the weakest models gain nothing from a better harness
even when they can write one.

## Self-Harness: propose, validate, accept

**Self-Harness** (Zhang et al. 2026) runs the loop with an explicit accept/reject gate.

*Weakness mining.* Evaluate under the current harness $h_t$ and collect execution traces.
The key observation is that **the verifier outcome is not the failure**: two runs both
logged as "timeout" or "missing artifact" can have entirely different causal mechanisms.
So a failure record carries three layers — the terminal verifier-level cause, the causal
status of the relevant agent behavior, and the abstract mechanism the trace exposes —
and failures are clustered into *verifier-grounded patterns* rather than error strings.

*Harness proposal.* The same model, running under $h_t$, proposes edits from a bounded
context: the editable surfaces, the mined failure patterns, records of passing behaviors
**that must be preserved**, and summaries of edits already tried. Edits should target
recurrent and addressable patterns — not task-specific difficulty — and candidates should
be diverse.

*Proposal validation.* Every candidate is regression-tested on a held-in split (did the
weakness resolve?) and a held-out split (did anything else break?), and accepted **only
with no regression on both**. Rejected candidates are logged without touching the active
harness.

On Terminal-Bench-2 with MiniMax M2.5, Qwen3.5-35B-A3B and GLM-5, the loop learned
*model-specific* harness instructions — different base models have different weaknesses,
so the harness that fits each differs. That result argues against a single canonical
harness and for harnesses as model-conditional artifacts.

## AHE: the bottleneck is observability

**Agentic Harness Engineering** (Lin et al. 2026) diagnoses the failure of naive harness
evolution as an attribution problem: when a rollout fails, which component is responsible?
Its answer is three pillars.

**Component observability.** Every editable component has a file-system representation, so
the action space is explicit and traceable. AHE enumerates seven: system prompt, tool
description, tool implementation, middleware, skill, sub-agent configuration, long-term
memory. Each failure pattern maps to one component, which is what makes an edit targeted
rather than a shotgun.

**Experience observability.** A hierarchy, for token efficiency. Each harness generates
$k$ traces, one file each; an "agent debugger" writes a per-task root-cause report; the
reports aggregate into a benchmark overview. The evolving agent reads the overview, and
descends to raw traces only when it needs to. This is the same compile-at-ingest economics
as the [[llm-wiki-pattern]] — pay once to summarize, read the summary many times.

**Decision observability.** Every edit is a **file-level, falsifiable claim**, paired with
a prediction verified next round. Its manifesto entry names the failure evidence, the
inferred root cause, the targeted fix, and a predicted impact covering both expected fixes
and at-risk regressions. The read-only constraints that make the attribution honest are on
[[harness-reward-hacking]].

AHE beat human-designed harnesses (OpenCode, Terminus-2, Codex) on Terminal-Bench-2
outside the Hard tier. The result that matters more: the **frozen** evolved harness, with
no further evolution, transferred to SWE-bench-verified — evidence that it encoded
engineering experience into components rather than overfitting the benchmark.

## Evolutionary search over harnesses

Evolution fits when the search space is large or oddly shaped and solutions are hard to
optimize by gradient but easy to *evaluate*. Harness search qualifies.

The lineage runs through prompts first — **Promptbreeder** (Fernando et al. 2023), where
the mutation prompts themselves evolve, and **GEPA** (Agrawal et al. 2025), which reflects
in natural language over trial-and-error trajectories to propose updates — and then to
programs with **AlphaEvolve** (Novikov et al. 2025): a pool of candidate programs, frozen
LLMs generating diffs, survivors kept. Three design details carried forward:

- The prompt includes parent programs, their results, instructions, and meta-information.
- The agent sees the full repo, but improvable regions are explicitly fenced with
  `# EVOLVE-BLOCK-START` / `# EVOLVE-BLOCK-END`.
- The **meta-prompt co-evolves** alongside the instructions and context.

Variants: **ThetaEvolve** (Wang et al. 2025) adds RL and in-context learning;
**DemoEvolve** (Che et al. 2026) seeds the archive with human expert demonstrations;
**ShinkaEvolve** (Lange et al. 2025) attacks sample efficiency three ways — parent
sampling balancing performance rank against offspring count, embedding-cosine rejection
of candidates too similar to the population, and a meta-scratchpad recording what worked.
The middle one is an explicit countermeasure to diversity collapse.

**Darwin Gödel Machine** (Zhang et al. 2025) is the one aimed squarely at harness code:
an agent permitted to modify its own repository. Start with one agent; each iteration
picks a parent with probability proportional to performance and *inversely to its number
of children* — the same anti-collapse pressure — has it read its own benchmark evaluation
log and propose improvements to its own harness codebase, using just two tools (`bash`,
and an `editor` with `view/create/edit`), then evaluates and admits the good ones. Under a
fixed `Claude 3.5 Sonnet` and a simple starting harness, discovered agents went 20% → 50%
on SWE-bench Verified and 14.2% → 30.7% on Polyglot, matching or beating handcrafted
agents. Hyperagents (Zhang et al. 2026) adds a meta-agent controlling how task agents are
modified.

The boundary condition is honest and sharp: this family works where fitness is
automatically and cheaply quantifiable — matrix multiplication, GPU kernels, algorithm
contests, datacenter scheduling — and struggles wherever evaluation is slow, ambiguous, or
heuristic. See [[auto-research-evaluation]].

## Optimizing weights at the same time

Harness evolution changes the non-parametric system around a fixed model. **SIA** (Hebbar
et al. 2026) lets both move: a Meta-Agent proposes the initial harness, a Task-Specific
Agent executes, and a **Feedback-Agent decides per iteration whether to update the harness
or the weights** based on recent trajectories. **Continual Harness** (Karten et al. 2026)
does something similar in long-horizon gameplay, co-learning the policy by distilling a
strong teacher's labels on low-reward trajectories.

Weng treats this direction as interesting but the evidence as provisional — SIA's task
agent (`gpt-oss-120b`) is much weaker than its meta and feedback agents
(`Claude Sonnet 4.6`), and its baselines are too weak to cross-reference. Training
stability and Goodhart effects remain open.
