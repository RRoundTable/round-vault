---
type: source
tags: [decision-models, calibration, structured-output, routing]
category: machine-native-models
summary: "TypeSafe's docs: the Choice/Score/Noul contract, why questions run in isolation, how 'confidence' is computed from the probabilities (not measured separately), risk-scaled thresholds, and the four usage patterns."
raw: raw/2026-09-23-typesafe-docs.md
url: https://docs.typesafe.ai/introduction
---

# TypeSafe docs

Five pages of TypeSafe's developer docs (Introduction, System One, AI primer, Confidence,
Patterns), captured 2026-09-23 as the raw markdown the site serves. They specify what the
launch post only gestured at: the exact output contract of [[jev]], and how its confidence
numbers are produced and meant to be used. The Confidence page is the most informative.
It shows that **`confidence` is computed from the returned probabilities, not estimated
separately**, and it moves threshold-setting onto the user's own data. That makes it the
clearest statement of what "calibrated decisions" is supposed to buy a program, and of how
much of the work is left to the integrator. The docs are changing. Several URLs quoted in
third-party coverage no longer resolve.

## The contract

A request is a **state** (text, JSON objects or arrays of text; no images or audio) plus
named **questions**, each one of three primitives:

| Primitive | Asks | Returns |
| --- | --- | --- |
| Choice | pick one of your options | the choice, `probabilities` over options, `confidence` |
| Score | rate against ordered levels you define | a continuous score (e.g. `1.4` on 0–2), the distribution, `confidence` |
| Noul | is this statement true | a single probability, **no `confidence` field** |

All questions in a call are evaluated **in parallel and in isolation** against the same
state. Two consequences are claimed: adding questions "barely changes the response time",
and since no question sees another's answer, adding more "does not create context-rot."
The flip side, which the docs leave unstated, is that nothing inside a call can depend on
another answer. Any chaining happens in the caller's code across calls.

The endpoint is `POST /v1/systemone` with `model: jev-latest`. Every page opens with a
pointer to an `llms.txt` index, so the docs are formatted for agents to read as well as
people.

## Atomic questions, composed in code

The usage rule is stated as design guidance, not a tip. Each question should be a
"gut-check determination: the kind of judgment a highly knowledgeable person could make in
a few seconds given the right context." Anything needing extended reasoning or weighing
independent factors should be split, with the factors asked separately and "combined with
logic in your code." The docs present this as a feature ("when priorities shift, change a
coefficient in your code rather than rewriting a prompt"). It is also a constraint: the
model is not meant to do the combining.

## Confidence is a statistic of the probabilities

`confidence` "is a statistic computed from the probability distribution the answer already
gives you." It is 1.0 when all the mass is on one option and falls as the mass spreads.
The interactive demo on the page approximates it for *n* options as
`(n · p_max − 1) / (n − 1)`, a normalized top-option probability. TypeSafe calls it "a
solid default" and hands over the full distribution so the user can compute something
else. For a Score, low confidence may mean the levels are "ambiguous, multi-dimensional,
or the state doesn't contain enough to go on". A flat distribution can reflect a badly
posed question as well as genuine uncertainty.

What follows, and the docs don't say it: **confidence carries no information beyond the
probabilities.** If the probabilities are calibrated, confidence is useful. If they are
not, confidence inherits the error. Every claim about trustworthy confidence is therefore
a claim about calibrated probabilities. See [[calibration]].

## How software is meant to use it

The Confidence page proposes three bands:

- **high**: act automatically
- **medium**: proceed with caution, meaning confirm, flag for review, or gather more
  information
- **low**: don't act; route to a human, ask for clarification, or fall back to another
  system

Thresholds **scale with the stakes of the action**, not with the question. The worked
example uses a 0.5 floor below which everything goes to a human. A read-only action runs
above the floor, and a money transfer needs >0.9 before it proceeds without extra
confirmation. The note under it moves the work back to the user: "The correct threshold
values depend on your domain and the performance of the model for your use case. Start
with conservative thresholds, test with your own data, and adjust."

The Patterns page names four compositions: **speculative fan-out** (ask many questions,
including speculative ones, in one call and let code pick what's relevant),
**confidence-gated routing**, **composite scoring**, and **intent routing**. See
[[confidence-gated-routing]].

## The positioning, in their words

The AI primer is the manifesto. Its premise is that large-scale automation will be "closer
to 99% machine-to-machine interactions and 1% human interaction," so the machine interface
matters more than the chat interface. The goal it names, "Machine Native Intelligence", is
AI with software properties: "structure, reliability, observability, testability, speed,
consistency, and low cost". The slogan is "Building prod, not God."

Its case against RLHF has two parts. First, preference optimization rewards sycophancy
and "confident-sounding hallucinations". Second, it causes **mode dropping**, a milder
form of GAN mode collapse in which the model concentrates probability on one style and
loses the rest of the distribution. The docs define calibration correctly and modestly:
it "is measured across groups of predictions; it does not guarantee that an individual
answer is correct." Neither the Confidence page nor the primer reports a calibration
measurement.
