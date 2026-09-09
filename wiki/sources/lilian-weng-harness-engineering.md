---
type: source
tags: [harness, self-improvement, agent-workflow, context-engineering, evaluation]
category: agent-architectures
summary: "Weng's survey arguing recursive self-improvement arrives first through the harness, not the weights: design patterns, the optimization ladder, and seven bottlenecks."
raw: raw/2026-09-09-lilian-weng-harness-engineering.md
url: https://lilianweng.github.io/posts/2026-07-04-harness/
---

# Lilian Weng — Harness Engineering for Self-Improvement

A survey organizing recent work on auto-research, self-improving agents, and evolutionary
program search around one claim: the layer between a raw model and the real world matters
about as much as the model's raw intelligence, and it is the layer that will improve
first. Weng's framing is that **recursive self-improvement is unlikely to begin with a
model rewriting its own weights** — it begins with a model rewriting its [[harness]].

The post is a map of a literature, not a single result. Its value here is the taxonomy
and the failure analysis, not any one system.

## What a harness is

> the system surrounding a base model that orchestrates execution and decides how the
> model thinks and plans, calls tools and acts, perceives and manages context, stores
> artifacts, and evaluates results.

Weng contrasts this with her own 2023 formulation, "agent = LLM + memory + tools +
planning + action". A harness adds *workflow design, evaluation, permission control, and
persistent state management* — moving the subject from prompt templates to runtime and
software-system design. She draws an explicit analogy to an operating system: encapsulate
complicated logic, keep the interface simple, expect configs and tool protocols to
standardize across the industry.

Three recurring design patterns, and a case study on coding agents, are covered on
[[harness]].

## The progression of what gets optimized

The organizing spine of the post:

> instruction prompts → structured context → workflow → harness code → optimizer code

Each step trades a hand-written artifact for a searchable one, and each becomes viable
only as the base model gets strong enough to search it. The turn that matters is the
middle one: once a harness is *code*, an LLM can optimize it, and code is a universal
language for programs and systems, so the accessible design space is far larger than
prompts. Detail on [[self-improving-harness]]; the context-engineering rungs are on
[[agentic-context-engineering]].

## Systems surveyed

| System | Optimizes | Method |
| --- | --- | --- |
| ACE (Zhang et al. 2025) | context | generator / reflector / curator, itemized bullets |
| MCE (Ye et al. 2026) | context *mechanism* | bi-level; meta-agent crossover over skills |
| Meta-Harness (Lee et al. 2026) | harness-optimizing code | coding-agent proposer, Pareto frontier |
| AI Scientist (Lu et al. 2026) | — (handcrafted) | idea → code → experiment → manuscript → review |
| ScientistOne (Meng et al. 2026) | — (handcrafted) | verifiability first; Chain-of-Evidence audits |
| Autodata (Kulikov et al. 2026) | training data | challenger / weak solver / strong solver / verifier |
| ADAS (Hu et al. 2025) | workflow code | meta-agent search over an archive |
| AFlow (Zhang et al. 2025) | workflow graph | MCTS over LLM-modified graphs |
| STOP (Zelikman et al. 2023) | the improver itself | recursive meta-utility |
| Self-Harness (Zhang et al. 2026) | harness | mine → propose → validate |
| AHE (Lin et al. 2026) | harness | three observability pillars |
| DGM (Zhang et al. 2025) | harness repo | evolutionary, agent edits its own code |
| SIA (Hebbar et al. 2026) | harness *and* weights | feedback-agent routes each update |

Weng is not uniformly credulous about these. She flags Autodata as closer to "indirect
distillation over a generated prompt distribution" than real RSI, because synthesized
tasks fine-tune the weak solver and never the strong one. On SIA she calls the evidence
"provisional": the task agent (`gpt-oss-120b`) is far weaker than the meta and feedback
agents (`Claude Sonnet 4.6`), and the baselines are too weak to cross-reference.

## The cautionary results

Two findings do more work than the systems do, and both are about the base model.

**STOP improved downstream performance with GPT-4 and *degraded* it with GPT-3.5 and
Mixtral.** Recursive structure alone is not enough; the model must be capable enough to
improve the mechanism. Weng's conclusion: harness improvement enables better deployment
of a model, but intelligence is still the core.

**Lin et al. (2026) split the capability in two** and find the halves behave differently
— *harness-updating* is roughly flat from Qwen3.5-9B to Claude Opus 4.6 (the 9B proposer
writes a skill "procedurally isomorphic" to Opus's), while *harness-benefit* is
non-monotonic, peaking at middle-tier models. Writing a good harness edit is easy;
exploiting one requires correct and timely tool invocation and long-horizon instruction
following. Discussed on [[self-improving-harness]].

## Where an agent editing its own harness goes wrong

Weng raises the objection directly: if a program may edit the OS, abstraction boundaries
break. Her prescription — the editable surface must be designed, and permission control
and security layers must live *outside* the loop — is the core of
[[harness-reward-hacking]], along with AHE's read-only enforcement and Self-Harness's
two-split acceptance rule.

## Seven bottlenecks to full RSI

1. **Weak and fuzzy evaluators.** Self-improvement works where metrics are objective.
   Research taste — problem framing, experimental design, judging which surprising result
   is worth pursuing — has no fast verifier.
2. **Context and memory lifecycle.** Memory grows with autonomy. Weng argues context
   engineering "will and should become a core part of intelligence, rather than staying
   in the software system layer."
3. **Negative results.** The literature is biased toward successes, so models trained on
   it are bad at abandoning a hypothesis or reporting a failure. A research harness should
   make failed attempts easy to preserve.
4. **Diversity collapse.** Evolutionary and RL loops exploit known high-reward patterns.
   Critical for open-ended research, where the best path initially looks worse under the
   current evaluator.
5. **Reward hacking.** The loop optimizes whatever signal it is given — unit tests, a
   judge model, or benchmark artifacts.
6. **Long-term success.** Coding agents complete the task at hand; sandbox RLVR training
   rarely captures maintainability, ownership boundaries, migration cost, backwards
   compatibility, or future debugging burden.
7. **The role of humans.** "Humans should move up the stack, not be removed from the
   loop" — oversight at the right time and the right abstraction level, with touchpoints
   designed in.

Bottleneck 1 and the benchmark appendix are covered on [[auto-research-evaluation]],
which also carries Trehan & Chopra's six observed failure modes; bottleneck 5 is
[[harness-reward-hacking]].

## Note on this capture

`raw/` holds an HTML-to-markdown conversion of the post. Prose, equations, tables, and
links are faithful; the 14 figures survive as `![](name.png)` references with no image
files behind them, and figure captions are preserved as italic lines. Nothing on the
wiki pages depends on reading a figure.
