---
type: page
tags: [evaluation, benchmarks, auto-research, self-improvement, harness]
category: evaluation
summary: "Self-improvement loops only work where the verifier is fast and precise, which excludes most of research — the benchmarks that exist, and the failure modes they miss."
---

# Auto-research evaluation

Every harness self-improvement method depends on being able to score a candidate. That
makes evaluation the binding constraint on the whole program, not a reporting detail:
evolutionary and propose-validate loops work well where fitness is cheap and objective —
matrix multiplication, GPU kernels, algorithm contests, datacenter scheduling — and
degrade wherever evaluation is slow, ambiguous, or heuristic
([[lilian-weng-harness-engineering]]). Research is mostly the second kind.

## The fuzzy-evaluator problem

Weng lists weak and fuzzy evaluators first among the bottlenecks to recursive
self-improvement. Current loops work best when metrics are measurable and objective, for
the same reason RL does. What has no fast verifier:

- **research taste** — which mixes problem framing, experimental design, and judgment
  about which surprising result is worth chasing and which failure is worth a retry
- **novelty**
- **long-term scientific value**

There is a second-order version of the problem that is easy to miss. A harness loop needs
to score not just solutions but *itself*, and the thing it most wants to know — did this
edit encode a general mechanism or a benchmark-specific patch? — is exactly what a single
benchmark cannot answer. This is why held-out splits and transfer results carry
disproportionate weight in this literature; see [[harness-reward-hacking]].

## Paper production is not discovery

The AI Scientist line of work shows an expert-designed harness can coordinate much of an
auto-research loop, measured by producing papers. Weng's objection is that the metric and
the goal come apart: **a system can write a plausible manuscript while carrying fabricated
citations, implementation drift, or weak results.**

Trehan & Chopra (2026) tested idea-to-paper with minimal scaffolding — `read_file`,
`write_file`, `llm_search`, `list_files`, one workspace per idea — across world models,
multi-agent RL, and AI safety, each seeded with 45–50 high-quality documents. The scale of
the result is itself the finding: of the ideas generated, human experts selected four to
run through the full pipeline, and **one** was executed into a paper. Six recurring failure
modes:

| Failure mode | What it looks like |
| --- | --- |
| Bias toward training-data defaults | old libraries, stale commands, assumptions not grounded in the actual repo or dataset |
| Implementation drift under execution pressure | when the proposed method gets technically hard, quietly substitute the common simpler one |
| Memory and context degradation | long-horizon projects lose critical details unless logs are written as persistent artifacts |
| Over-optimism | declare success on noisy or failed experiments |
| Insufficient domain intelligence | can't predict implementation complexity, judge whether a result is plausible, or know which baselines matter |
| Weak scientific taste | experiments run fine and answer the wrong question |

Two of these are harness problems with known fixes. Memory degradation is what
file-system-as-persistent-memory exists for ([[harness]]), and it is the clearest case in
this literature of a failure mode that is *architectural* rather than a capability gap.
Implementation drift is what ScientistOne's verifiability-first design targets, requiring
every claim — citation, numerical, methodological, conclusion — to trace to an evidence
source, audited by Chain-of-Evidence checks.

The other four are not harness problems. Over-optimism is the one to watch, because it is
the failure mode that *corrupts the evaluator*: Bubeck et al. (2025) describe the same
pattern as "p-hacking and eureka-ing", where models apply "numerical duct tape" and declare
victory while the signal is still noise. A loop whose scorer is prone to this cannot be
fixed by adding more of it.

## Benchmarks

| Benchmark | Tasks | Best reported result |
| --- | --- | --- |
| **PaperBench** | replicate 20 ICML 2024 Spotlight/Oral papers from scratch; 8,316 rubrics co-developed with the authors | `Claude 3.5 Sonnet` ~21%, below ML PhDs |
| **CORE-Bench** | 270 computational-reproducibility tasks from 90 papers across CS, social science, medicine | `GPT-4o` 21% on the hardest tier |
| **ScienceAgentBench** | 102 tasks from 44 publications in math, chemistry, biology, geography | — |
| **RE-Bench** | 7 open-ended ML research-engineering environments, ≤8 H100s each | see below |
| **MLE-bench** | 75 Kaggle ML-engineering competitions, graded against public leaderboards | `o1-preview` + AIDE, bronze in 16.9% |
| **KernelBench** | 250 PyTorch tasks for GPU kernel generation | scored by `fast_p` — correct *and* faster than baseline |

Design details that matter: PaperBench decomposes each replication into individually
gradable subtasks and ships JudgeEval to evaluate its own grader. RE-Bench defines each
environment as a (scoring function, starting solution, reference solution) triple —
concrete tasks like optimizing a kernel, running a scaling-law experiment, fixing an
embedding, fine-tuning GPT-2 for QA.

## The RE-Bench crossover

RE-Bench's human comparison is the most useful single datapoint in the appendix, because it
has a *shape* rather than a number. It includes 71 eight-hour attempts by 61 distinct human
experts, who scored non-zero in 82% of attempts and matched or exceeded strong reference
solutions in 24%.

Against that: **the best AI agents scored 4× higher than humans at a 2-hour budget, but
humans had better returns to longer budgets and exceeded agents at 8 and 32 hours.**

Agents win the sprint and lose the marathon. This is the empirical form of the long-horizon
problem the harness literature is trying to solve — and it locates the deficit in
*returns to time*, not in raw skill. That is precisely what better context management,
durable artifacts, and failure analysis are supposed to buy ([[harness]],
[[agentic-context-engineering]]), which makes the crossover point the natural metric for
whether harness engineering is working. Two of Trehan & Chopra's six failure modes —
context degradation and over-optimism — are plausible mechanisms for why the curve
flattens.

## Negative results are missing from the training data

Weng's third bottleneck is an evaluation problem wearing different clothes. Researchers are
incentivized to publish successes, so the literature is biased toward them, so models
trained on it are bad at deciding when to abandon a hypothesis, report a negative result,
or acknowledge a failure at all. This is the likely upstream cause of the over-optimism
failure mode.

Her prescription is a harness requirement: **a research harness should make failed attempts
easy to preserve**, because learning from failure is the most efficient way to trim the
search space. The systems that do this treat failure as first-class data rather than an
error log — ACE's reflector distills insights from failed trajectories as well as
successful ones ([[agentic-context-engineering]]), and Self-Harness clusters failures into
verifier-grounded patterns where the *causal mechanism*, not the terminal error string, is
the unit of analysis ([[self-improving-harness]]).

The same argument applies to a knowledge base: a source that failed to support a claim, or
a page revised because it was wrong, is worth keeping legible. In this vault git carries
that — `git log -S'<claim>'` finds when a claim entered, and a revision leaves the
superseded version reachable, which an append-only summary of successes would not.
