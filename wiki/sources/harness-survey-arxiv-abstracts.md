---
type: source
tags: [harness, self-improvement, evolutionary-search, evaluation, bibliography]
category: agent-architectures
summary: "Verified abstracts for all 29 arXiv works Weng cites: the numbers, ablations and boundary conditions her survey compressed out, plus where it paraphrases loosely."
raw: raw/2026-09-09-harness-survey-arxiv-abstracts.md
url: https://arxiv.org/list/cs.AI/recent
---

# arXiv abstracts behind the harness survey

The 29 arXiv references in [[lilian-weng-harness-engineering]], fetched from the arXiv API
on 2026-09-09. **Every requested ID resolved and every title matched the survey's
description** — the survey's citations are accurate, which is worth knowing before leaning
on it. This page records what the abstracts add beyond the survey's paraphrase, and the
handful of places the survey's compression loses something.

**Scope limit, stated up front: this is abstracts and metadata, not full papers.** Claims
traced here are author-reported headline results. Numbers, ablations and stated boundary
conditions are reliable at this level; method detail, experimental caveats and anything
requiring a results table are not. Pages citing this source should not imply a closer
reading than happened.

## What the primaries add

**AHE** (arXiv:2604.25850) supplies numbers the survey omits and one finding it drops
entirely. Ten iterations lift Terminal-Bench 2 pass@1 from **69.7% → 77.0%**, past
Codex-CLI's 71.9%. The frozen harness transfers to SWE-bench-verified at **12% fewer
tokens** than the seed and yields **+5.1 to +10.1pp** cross-family gains on TB2 across
three other model families. The dropped finding is the most useful sentence in the abstract:
ablations **localize the gain to tools, middleware and long-term memory rather than the
system prompt**, so "factual harness structure transfers while prose-level strategy does
not." The abstract also calls components "revertible", not merely traceable — a stronger
property than the survey conveys.

**Self-Harness** (arXiv:2606.09498) ran on **three** benchmarks, not the one the survey
names: Terminal-Bench-2.0, SWE-bench Verified, and AppWorld — nine model–benchmark
combinations, all improving both held-in and held-out pass rates, with relative gains **up
to 132%**. The retained mechanisms are benchmark-specific bottlenecks (artifact handling
and runtime control; software-patch verification; application-state retrieval), which sits
in mild tension with the paper's own framing that harness design is *model*-specific — the
abstract asserts both. It also stresses what the survey does not: improvement happens
without human engineers **or stronger external agents**, so this is not distillation.

**Lin et al.** (arXiv:2605.30621) gives the mechanism behind the weak-tier result. Weak
models fail in two identifiable ways: they fail to *activate* the relevant harness
artifact, or they activate it and fail to *follow* it faithfully. Strong-tier models
benefit less than mid-tier. The prescription is explicit and actionable — **spend capability
budget on the task-solving agent rather than the evolver**, and train for harness
invocation and long-horizon instruction following. Code at
`github.com/A-EVO-Lab/a-evolve`.

**Meta-Harness** (arXiv:2603.28052) states a motivation the survey skips, and it is the
sharpest framing of the problem in the whole bibliography: existing text optimizers are
poorly matched to harness search because **they compress feedback too aggressively**. Results: +7.7 points over a state-of-the-art context
management system while using **4× fewer context tokens**; +4.7 points on 200 IMO-level
problems averaged across five held-out models. Author overlap matters here — Qizheng Zhang
appears on both ACE and Meta-Harness, and Omar Khattab (DSPy) and Chelsea Finn are
co-authors, placing this in the program-optimization lineage rather than the agent lineage.

**ACE** (arXiv:2510.04618) reports +10.6% on agents and +8.6% on finance with reduced
adaptation latency and rollout cost, and two things the survey leaves out: it adapts
**without labeled supervision**, using natural execution feedback, and on the AppWorld
leaderboard it matches the top production-level agent overall while beating it on the
harder test-challenge split — using a *smaller open-source model*.

**MCE** (arXiv:2601.21557) reports 5.6–53.8% relative improvement over prior agentic CE
methods across five domains (mean 16.9%), and frames itself as an explicit critique of
ACE's shape: "rigid generation-reflection workflows and predefined context schemas" that
"impose structural biases and restrict context optimization to a narrow, intuition-bound
design space."

**DGM** (arXiv:2505.22954) is clearer than the survey about the conceptual move. Schmidhuber's
Gödel machine required *proving* each self-modification beneficial, which is impractical;
DGM substitutes **empirical validation on benchmarks** for proof. The abstract also names
what DGM actually discovered — better code editing tools, long-context window management,
peer-review mechanisms — and notes all experiments ran with sandboxing and human oversight.

**Hyperagents** (arXiv:2603.19461) deserves far more than the survey's one line, because it
identifies the hidden assumption in DGM: DGM works in coding *because evaluation and
self-modification are both coding tasks*, so coding gains convert directly into
self-improvement gains — **and that alignment does not hold outside coding**. Hyperagents
merges task agent and meta agent into one editable program in which the modification
procedure is itself editable. Its meta-level improvements (persistent memory, performance
tracking) **transfer across domains and accumulate across runs**.

**ThetaEvolve** (arXiv:2511.23473) is evidence for the survey's own internalization
prediction, which the survey does not connect. Its stated complaint about AlphaEvolve is
that it "is a pure inference system that models cannot internalize the evolving strategies."
With test-time RL, checkpoints show **faster progress and better final performance on
unseen tasks** — the evolving capability moves into the weights. It is also the first to get
a small open-source model (DeepSeek-R1-0528-Qwen3-8B) to new best-known bounds on circle
packing and the first autocorrelation inequality, using a single LLM rather than frontier
ensembles, with lazy penalties against stagnant output.

**DemoEvolve** (arXiv:2605.24539) is a boundary condition, not just an archive tweak.
Self-rollout harness evolution works when episodes are short and failures attributable
(Liar's Dice) and **breaks in long-horizon stochastic regimes** (Balatro), where sparse
reward and candidate-selection noise actively mislead the search — and tutorial-like
textual knowledge alone does not rescue it. Human demonstrations do.

**Continual Harness** (arXiv:2605.09998) buries a notable result. Its Gemini Plays Pokemon
experiments, with human-in-the-loop harness refinement, were **the first AI system to
complete Pokémon Blue, Yellow Legacy on hard mode, and Crystal without a lost battle** —
and in the hardest stages the agent began iterating on its own strategy through long-context
memory. Continual Harness then removes the human and is **reset-free**: it adapts online
within a single run, where prompt-optimization methods require episode resets. From a raw
interface with no domain scaffolding it recovers a majority of the gap to a hand-engineered
expert harness.

**SIA** (arXiv:2605.27276) reports SIA-W+H beating scaffold-iteration-alone on all three
domains: **25.1%** over prior SOTA on LawBench, **12.4%** faster GPU kernels (1,017 vs
1,161 μs), **20.4%** over SOTA on single-cell RNA denoising. Its one-line division of labor
is cleaner than the survey's: "harness updates make the model agentic, shaping how it
searches and acts, while weight updates build the domain intuition that no prompt or
scaffold can instil."

**ScientistOne** (arXiv:2605.26340) is a much larger evaluation contribution than
"verifiability is the design constraint." It ships **CoE Audit**, four post-hoc integrity
checks — score verification, specification violation, reference verification, method-code
alignment — applied uniformly to all systems. Across 75 papers from five systems on five
tasks, **every baseline had at least one systematic failure mode**: hallucinated reference
rates up to **21%**, score verification passing in as few as **42%** of papers, method-code
alignment ranging **20–80%**. ScientistOne itself: 0/337 hallucinated references, 12/12
score verification, 14/15 method-code alignment.

**Trehan & Chopra** (arXiv:2601.03315) — real title *Why LLMs Aren't Scientists Yet* — used
a pipeline of **six** LLM agents across four attempts; three failed during implementation or
evaluation. The one that completed was **accepted to Agents4Science 2025**, a venue
requiring AI systems as first authors, passing human and multi-AI review. The paper's own
name for the over-optimism mode is "overexcitement". Artifacts released at
`github.com/Lossfunk/ai-scientist-artefacts-v1`.

**Autodata** (arXiv:2606.25996) names its implementation **Agentic Self-Instruct** and
tests on CS research, legal reasoning, and mathematical-object reasoning. Its emphasized
result is that **meta-optimizing the data scientist agent itself delivers a larger uplift**
than the base method — which bears on the survey's critique (see below).

## Where the survey's compression costs something

- **Autodata.** Weng judges it "closer to indirect distillation over a generated prompt
  distribution" because synthesized tasks fine-tune the weak solver and never the strong
  one. The abstract's headline is a different axis — meta-optimizing the *data scientist*,
  not the solver — and on that axis there is a real recursive loop. Her critique may still
  hold for the solver loop; the abstract is not enough to settle it, and the two claims are
  about different components. Flagged rather than resolved.
- **Self-Harness scope.** Presented as a Terminal-Bench-2 result; it is three benchmarks.
- **AHE's ablation.** The tools/middleware/memory-versus-system-prompt finding is absent
  from the survey, and it is arguably the most transferable practical lesson in it.
- **Hyperagents.** One line for a paper whose argument reframes what the DGM result means.
- **Figure caption drift.** The survey's own figure caption for Lin et al. says
  "Qwen2-32B to Opus 4.6" while its body text and the paper say Qwen3.5-9B. Body text is
  right.
- **Bubeck et al.** is cited for "p-hacking and eureka-ing"; the actual paper is *Early
  science acceleration experiments with GPT-5* (arXiv:2511.16072), a broader work.

## Lineage worth noting

The authorship graph is small and tells you the ideas are converging rather than
independently invented. Shengran Hu (ADAS) → Jenny Zhang, Jeff Clune (DGM) → Hyperagents
is one continuous line, with Robert Lange bridging DGM to ShinkaEvolve. Qizheng Zhang
connects ACE to Meta-Harness, which is also Khattab-and-Finn territory. So the two apparent
schools — evolutionary agent search out of Clune's group, and program/context optimization
out of the DSPy lineage — meet at harness code as the shared object.
