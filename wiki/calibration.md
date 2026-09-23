---
type: page
tags: [calibration, decision-models, evaluation]
category: machine-native-models
summary: "What calibration means (a group property, not a per-answer guarantee), why it is the property that lets software act on a model's output unattended, and why a confidence score adds nothing if the probabilities under it are not calibrated."
---

# Calibration

A model is **calibrated** when its stated probabilities match observed frequencies: of all
the answers it gives probability 0.8, about 80% are right. It is a property of *groups* of
predictions and guarantees nothing about any single answer ([[typesafe-docs]]). For a
human reader, calibration is a nicety. For a program acting unattended, it is **the
interface**. Code cannot read tone or hedging. It can only compare a number to a
threshold, and that comparison is meaningful only if the number is calibrated. This is
the core of the machine-native case [[jev]] makes: a model built for software consumers
has to be trained for honest probabilities, not for preferred text.

## Why software needs it

A model that does a task correctly 95% of the time but gives no sign of *which* 5% it gets
wrong cannot automate that task ([[typesafe-jev-launch-post]]). Every unattended decision
needs a way to tell the cases it can act on from the cases it should hand off. With
calibrated probabilities that is a threshold. Without them the choice is to review
everything or trust everything.

This is also why type safety is not enough. Constraining output to a schema removes parse
failures and invented values, but a schema-valid answer given with false confidence fails
the program as badly as a hallucinated string does. The question that matters for
unattended use is whether the wrong answers come with low confidence.

## Confidence scores are downstream of the probabilities

TypeSafe's `confidence` is computed deterministically from the returned probability
distribution. Its demo approximates it as a normalized top-option probability,
`(n · p_max − 1) / (n − 1)` ([[typesafe-docs]]). This generalizes. Any confidence score
derived from a model's own distribution **carries no information beyond that
distribution**. It makes thresholding convenient and does nothing for trustworthiness. If
the probabilities are miscalibrated, the confidence score is equally wrong, and there is
nothing separate to check.

There is a second trap. A flat distribution can mean the model is uncertain, or that the
question itself is ambiguous or multi-dimensional. TypeSafe says so for Score questions
([[typesafe-docs]]). Low confidence is therefore partly a signal about how the question is
posed, not only about the model's knowledge.

## Thresholds are per domain, and need labeled data

Even for a well-calibrated model, TypeSafe's guidance leaves the thresholds to the user:
set them by the stakes of the action, start conservative, and "test with your own data"
([[typesafe-docs]]). Calibration measured on the vendor's distribution does not carry over
to yours. So a decision model does not remove the need for labeled examples. It moves them
from training a model to setting its thresholds. See [[confidence-gated-routing]] for the
patterns that consume those thresholds.

## What training does to it

TypeSafe's position is that RLHF damages calibration in two ways. It rewards
confident-sounding answers raters prefer, and it causes *mode dropping*, collapsing the
output distribution toward one style so that the probabilities stop reflecting the real
spread of plausible answers ([[typesafe-docs]]). Its claimed remedy, RLCD, optimizes
probabilities against outcomes, but it is unpublished, and TypeSafe reports no calibration
measurement for Jev ([[typesafe-jev-launch-post]]).

The published record supports the premise at every stage of the pipeline
([[jev-related-arxiv-abstracts]]):

| Stage | Effect on calibration |
| --- | --- |
| pretraining | large models are well calibrated on multiple-choice and true/false questions in the right format |
| RLHF | conditional probabilities degrade; verbalized confidence becomes the better signal, by ~50% relative ECE |
| RLVR, binary rewards | guessing goes unpenalized; models become over-confident in wrong answers |

So the probabilities most worth thresholding on exist *before* post-training and are
damaged by it. That is the strongest version of TypeSafe's argument, and it comes from
other people's papers.

## Training for it directly

The fix the literature converges on is to reward a **proper scoring rule**, a score
minimized only when stated confidence equals the true probability of being right. RLCR
adds a Brier-score term to the correctness reward and proves that any bounded proper
scoring rule yields a model that is both accurate and calibrated. Empirically it improves
calibration with no accuracy loss, out of domain as well. Rewarding Doubt does the same
with the log score ([[jev-related-arxiv-abstracts]]). These are the nearest published
counterparts to RLCD, which is described in one sentence and unpublished.

Two further results shape what to expect from a model built this way
([[jev-related-arxiv-abstracts]]):

- **Accuracy and calibration pull against each other.** DCPO finds a gradient conflict
  between them under RLVR and fixes it by decoupling the objectives. Calibration needs its
  own objective, not a penalty term added to someone else's. This is the best support for
  building a separate model class around it.
- **Calibration is separable from accuracy.** A 4B model trained with proper scoring
  rules matched frontier models' calibration on factual QA while being much less accurate.
  A model can know *when* it is likely wrong without being right more often. That is
  exactly the property [[confidence-gated-routing]] needs from a cheap first stage.

The baseline any such claim has to beat is cheap. Temperature scaling, one parameter fit
after training, is "surprisingly effective" at calibrating neural classifiers
([[jev-related-arxiv-abstracts]]). A trained-in calibration result that doesn't report
against post-hoc scaling hasn't shown its training method was needed.

## Calibration does not remove hallucination

Kalai & Vempala show that a calibrated *generative* model must hallucinate arbitrary facts
at about the rate of facts seen once in training. Kalai et al. trace hallucination to
binary-classification errors, sustained by evaluations that reward guessing over abstaining
([[jev-related-arxiv-abstracts]]). Calibration therefore does not make wrong answers go
away. It makes them *priced*: a calibrated model is wrong at the rate its probabilities
say. For software that is the useful property, and it is the honest replacement for
"can't hallucinate" ([[jev]]).
