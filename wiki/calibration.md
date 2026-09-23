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
