---
type: page
tags: [decision-models, calibration, structured-output, classification]
category: machine-native-models
summary: "What Jev is (a typed, probabilistic decision model, not an LLM), which of TypeSafe's claims are verifiable and which are self-graded, and why decomposing questions is the price of using it."
---

# Jev

**Jev** is TypeSafe AI's first "System One model", released in early access on
2026-09-15. It is a model that **never generates text**. You send it a *state*
(unstructured context) and a set of typed questions, and it returns typed answers with
probabilities in a single parallel pass. Code consumes the answers directly, with nothing
to parse ([[typesafe-jev-launch-post]]). It is the clearest current instance of a model
designed for software, not a person, as the consumer, which is this wiki's subject. The
design argument is serious. The evidence so far is almost entirely self-graded: no paper,
no architecture disclosure, no public benchmarks by policy, and an eval that scores
agreement with frontier models rather than correctness.

## The contract

A request is a state plus named questions. Each question is one of three primitives
([[typesafe-docs]]):

| Primitive | Asks | Returns |
| --- | --- | --- |
| Choice | pick one of your options | choice, probabilities over options, confidence |
| Score | rate against ordered levels | continuous score, distribution, confidence |
| Noul | is this statement true | one probability (no confidence field) |

Questions in one call run **in parallel and in isolation** against the same state, so
adding questions barely adds latency and no question's context is polluted by another's
([[typesafe-docs]]). This is the sub-agent rule from [[harness]], applied inside a single
model call: independent work, kept separate so nothing contaminates the rest. The cost is
that nothing within a call can depend on another answer, so any dependency is expressed as
code across calls.

`confidence` is not a separate estimate. It is computed from the returned distribution,
approximately a normalized top-option probability, so it is exactly as trustworthy as the
probabilities under it ([[typesafe-docs]], [[calibration]]).

## What is claimed

TypeSafe's claims, grouped by how much the company itself stands behind them
([[typesafe-jev-launch-post]]):

| Claim | Status by TypeSafe's own account |
| --- | --- |
| 70–500 ms end-to-end | measurable per call; timed from their West Coast laptops |
| $0.042/MTok input, output free | published price; "can't prove it isn't subsidized" |
| never makes type errors | true by construction: outputs are confined to a declared schema |
| 0% hallucination | "not empirical", inferred from schema matching |
| similar intelligence to LLMs on System One tasks | self-built workflow evals, graded against frontier-model averages |
| 193.6× faster, 444.6× cheaper | same evals; "on the higher end of real world gains" |
| calibrated probabilities | asserted; no calibration measurement published |

The last row carries the whole product. Type safety alone is achievable with constrained
decoding on any LLM. What would make Jev a new category rather than a fast classifier
is that its probabilities are **honest**, meaning 0.8 is right 80% of the time. That is
the claim with the least evidence behind it.

## "Can't hallucinate" is a definition, not a result

Jev cannot produce an invented string because it produces no strings. It can still pick
the wrong option with high probability, and for a system acting on the answer that is the
same failure: a confident wrong output. TypeSafe's own chart labels its 0% as non-empirical
([[typesafe-jev-launch-post]]). The claim that would matter is *calibrated error*, where
confident answers are right and wrong answers come with low confidence. That is
unmeasured.

Hallucination theory sharpens this. Kalai et al. trace hallucinations to "errors in binary
classification" ([[jev-related-arxiv-abstracts]]). On that account a confidently wrong
Noul is not a way around hallucination. It is the elemental case. Kalai & Vempala do leave
an opening: pretraining gives "no statistical reason" to hallucinate *systematic* facts,
and other architectures may reduce those. That is a narrower claim than TypeSafe's, and a
testable one.

## The eval measures agreement, not accuracy

Jev's comparative evidence comes from "workflow evals". Every model runs the same fixed
workflow code, and answers are scored against the *average probabilities of GPT-6 Astra
and Fable 5.1* ([[typesafe-jev-launch-post]]). Two consequences follow:

- The eval can show Jev approaching the frontier models. It cannot show Jev exceeding
  them, and it cannot distinguish a calibrated model from one that imitates the
  references' miscalibration.
- It freezes the harness on purpose, to rule out gains from harness engineering. That is
  a defensible control, and it is the evaluator-side version of the concern in
  [[harness-reward-hacking]].

## Decomposition is the usage model

TypeSafe reports that one long prompt with the logic done in chain of thought performs
significantly worse than the same task split into many small questions and combined in
code ([[typesafe-jev-launch-post]]). Read plainly, the model is built to answer narrow
questions, and the *reasoning* moves out of the model and into the calling program. This
is the actual architectural bet. The model does System 1 judgments, and ordinary code
supplies the System 2 structure that an LLM would otherwise generate as tokens.

The docs make it a rule. A question should be a "gut-check determination" a knowledgeable
person could make in seconds. Anything that weighs several factors gets split, with the
parts combined by a formula in code ([[typesafe-docs]]). The pitch is control: reweight a
coefficient instead of rewriting a prompt. The unstated cost is that someone has to choose
those coefficients, and choosing them well takes labeled examples. The same holds for the
confidence thresholds, which TypeSafe tells users to tune on their own data. See
[[confidence-gated-routing]].

That puts Jev's natural home inside a [[harness]] rather than in place of the agent's
model. It suits the per-step decisions an agent loop makes over and over (route this,
gate that, is this done), where an LLM call is slow and expensive and a typed answer is
exactly what the loop needs. LangChain's integration does this, shipping Jev as a model
router and a pre-execution tool-risk gate ([[langchain-jev-harness]]). Neither middleware
uses confidence to escalate yet, so the property that would distinguish Jev from any fast
classifier goes unexercised in its first harness deployment.

## Where it sits in the literature

The published record supports Jev's premise and contests its novelty
([[jev-related-arxiv-abstracts]]).

**The premise holds.** Base models are well calibrated on multiple-choice and true/false
questions, which are Jev's formats. RLHF degrades that calibration, and binary-reward RL
makes models over-confident in wrong answers. Proper-scoring-rule rewards restore it
(RLCR, Rewarding Doubt), and DCPO finds accuracy and calibration gradients in direct
conflict. Together these are a better case for a calibration-first model class than
anything TypeSafe has published. See [[calibration]].

**The novelty is contested.** Zero-shot classification against arbitrary labels supplied
at call time was benchmarked and cast as entailment in 2019. Fine-tuned encoders like
ModernBERT are the production workhorse for fast structured classification. What Jev adds,
if its claims hold, is the combination: task specified in natural language per call,
frontier-level judgment, and trained calibration.

**The name is taken.** "RLCD" already refers to Reinforcement Learning from Contrastive
Distillation (2023), an unrelated alignment method.

**Decomposition has an explanation.** Format restrictions measurably degrade LLM reasoning,
and more so the stricter the format. A model emitting no reasoning tokens at all should
need questions small enough to require none. Work on distilling System 2 into System 1
names a boundary too: complex math reasoning that needs chain of thought did not distil.

## Known limits

- At most **255 options** per choice. Larger sets need a two-stage score-then-choose
  pattern.
- Text-only state. The Doom demo feeds game state as a text data structure.
- Speedups over LLMs shrink against non-reasoning modes. The Wikiracing demo compares
  against those and shows smaller gains.

([[typesafe-jev-launch-post]])
