---
type: source
tags: [decision-models, calibration, structured-output, evaluation, benchmarks]
category: machine-native-models
summary: "TypeSafe's launch of Jev: what a 'System One model' claims to be, the RLCD training pitch, the speed/cost numbers, and exactly which of them the company itself admits are non-empirical or biased."
raw: raw/2026-09-23-typesafe-jev-launch-post.md
url: https://typesafe.ai/blog/introducing-system-one-models-and-jev
---

# TypeSafe Jev launch post

Diogo Almeida's launch post (2026-09-15) introduces **Jev**, the first "System One model":
a model that gives up string generation entirely and returns only typed values with
probabilities, framed as "a frontier-intelligence function call: unstructured state in,
typed probabilistic decisions out." It claims two orders of magnitude of speed and cost
advantage over LLMs at similar intelligence on "System One tasks", and zero hallucination.
The post is unusually candid in its per-section "Nuance" notes. Read them and most of the
headline numbers turn out to be either self-graded or non-empirical by construction.
That candor makes it more useful than typical launch material. It is still the
vendor's own account, and it contains no paper, model card or architecture detail. See
[[jev]] for the synthesis.

## The pitch

Almeida's starting question: models have been "superhuman at chat for years, so where is
all the automation?" His answer is that the training objective is wrong for the job.
RLHF optimizes for the text a human rater prefers, which suits a chat product and not
automation. RLVR only fits tasks with cheap programmatic verification and "tends to cause
spikey / non-robust intelligence". TypeSafe's replacement is **RLCD, Reinforcement
Learning for Calibrated Decisions**, optimizing for "answers with epistemically honest
probabilities". The FAQ calls this "the bitterest lesson": optimizing for the right task
matters more than data, compute, or algorithms.

The stack is described in one sentence and never elaborated: "a new model architecture,
parallel sampler for maximum efficiency, and training method we call [RLCD]."

| | Existing LLMs | Jev |
| --- | --- | --- |
| Inputs | emphasis on sequential messages | emphasis on **structured program state** |
| Outputs | strings, needing parse + validate | typed values defined in advance; "never makes type errors" |
| Sampling | sequential, token by token | **parallel**, all outputs in one query |
| Price | $0.20–$10/MTok input, output ~5× | **$0.042/MTok input, output free** |
| End-to-end latency | 3–329 s | **70–500 ms** |
| Confidence | overconfident, inconsistent when asked | every output carries calibrated probabilities |

The claimed use cases are the useful part of the pitch because they say what shape of
work this is for. The post calls them "smart if-statements": classify, route, score,
extract, or branch where hand-written logic is too brittle. Also map-reduce over large
data, real-time applications at 100 ms, and "verify everything": scoring, judging,
guardrailing and jailbreak detection over LLM prompts, traces and outputs.

## What the post itself concedes

Each evidence section carries a "Nuance" note. Collected, they undercut most of the
headline:

- **Speed** is measured from the team's laptops on the US West Coast. **Cost**: "We can't
  prove it isn't subsidized."
- **The side-by-side demo** uses a simplified query and a "short, dense" state, which
  "paints our model in an advantageous light." The comparison model is GPT-5.6 Terra.
- **The 193.6× faster / 444.6× cheaper figures** come from their workflow evals and are
  "on the higher end of real world gains." The workflows were written by the model
  capabilities team, so "some bias could exist." The LLM baselines ran through TypeSafe's
  own structured-output wrapper, which "tends to be slower and more expensive" than asking
  those models for decisions without probabilities.
- **Hallucination at 0%**: "Our number is not empirical. Schema matching is guaranteed."
  The LLM comparison numbers come from OpenRouter, with acknowledged routing bias.
- **Public benchmarks**: deliberately not reported, and the company plans "only one-off
  evals when we make product updates."
- **Training data**: all made in-house and undisclosed ("if you want to find out more,
  we'd have to hire you").

## The workflow-eval design

This is the post's one methodological idea, and it is worth separating from the numbers
it produced. TypeSafe assumes the correct compute graph (the "workflow") is fixed code,
gives every model the same workflow, and scores each model against **reference
probabilities from the largest frontier models**, averaged from GPT-6 Astra and Fable 5.1.
Two consequences follow from that design, and the post states neither:

1. **It measures agreement with other models, not correctness.** A model can score
   perfectly only by matching the reference average. The eval cannot show Jev being *more*
   accurate than frontier models, and it cannot measure calibration against real
   outcomes, which is what RLCD claims to optimize.
2. **It deliberately freezes the harness.** Letting the harness vary is excluded as
   "potentially allowing for overfitting via harness engineering." This is the same worry
   [[harness-reward-hacking]] raises about self-improving loops, applied from the
   evaluator's side. See [[auto-research-evaluation]].

The post also reports that giving an LLM the whole task in one prompt, with the logic in
its chain of thought, does "significantly worse" than running the decomposed workflow.
That is the post's only claim about *decomposition*, and it becomes central in [[jev]].

## Demos and limits disclosed there

- **Doom**: 10 queries/s costs about $7/hour. The state is a text data structure, not
  images, and "a non-AI doom bot could play better."
- **Wikiracing**: Jev supports **at most 255 options** per choice. Larger choices run as a
  two-stage process, scoring each link independently and then choosing. The LLM baselines
  ran in non-reasoning modes.

## Naming

"System One" comes from Kahneman's fast/slow distinction, with the explicit caveat that
System 1 usually implies error-prone, while TypeSafe claims these models can be *more*
reliable. "Jev" is named for William Stanley Jevons: cheaper intelligence should grow
total demand, the way efficient steam engines did for coal.
