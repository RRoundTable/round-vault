---
type: page
tags: [routing, calibration, agent-workflow, decision-models]
category: agent-architectures
summary: "Using a model's confidence as a second decision axis: act, confirm, or escalate, with thresholds set by the stakes of the action. The patterns, and the calibration assumption every one of them rests on."
---

# Confidence-gated routing

**Confidence-gated routing** treats a model's certainty as a second decision axis next to
its answer. A confident answer is acted on automatically. An uncertain one is confirmed,
reviewed, or escalated to a slower system: a person, a reasoning model, or a stronger
model ([[typesafe-docs]]). It is the control-flow pattern that makes a cheap, fast model
safe to put in a loop, because the loop never has to trust any single answer
unconditionally. Every version of it rests on one assumption: that the confidence is
[[calibration|calibrated]]. Without that, the gate still routes, just not by likely
correctness.

## Three bands, thresholds by stakes

TypeSafe's reference version divides confidence into three ranges ([[typesafe-docs]]):

| Band | System behavior |
| --- | --- |
| high | act automatically |
| medium | proceed with caution: confirm with the user, flag for review, gather more information |
| low | don't act: route to a human, ask for clarification, fall back to another system |

The design point is that **thresholds belong to actions, not to questions**. In the worked
example, one intent classifier gates a read-only balance check and a money transfer
differently. A single 0.5 floor sends everything below it to a human. Above the floor, the
balance check runs, but the transfer needs >0.9 to proceed without extra confirmation. The
code encodes risk tolerance, so a destructive action and a recoverable one can share a
model and not a threshold.

## The pattern family

TypeSafe names four patterns for composing decisions ([[typesafe-docs]]):

- **Confidence-gated routing**: the above.
- **Intent routing**: classify what the user is trying to do and dispatch to a handler.
- **Composite scoring**: ask several narrow questions and combine them into one score in
  code.
- **Speculative fan-out**: because questions in a call run in parallel and in isolation,
  ask many at once, including ones that may turn out irrelevant, and let code decide which
  answers to use.

Speculative fan-out exists only because of the cost model. When extra questions add
almost no latency, it is cheaper to ask everything up front than to branch and call again.

## Where it sits in an agent

An agent [[harness]] makes the same small decisions over and over: which model should
handle this step, is this tool call safe, is this task done, does this output need review.
Each is a natural confidence-gated call. A fast decision model answers, high confidence
proceeds, and low confidence escalates to the expensive model or a person. The escalation
path is what makes the cheap first stage acceptable, and it only works if low confidence
reliably marks the cases the cheap stage gets wrong.

## What it cannot fix

A gate is only as good as the calibration behind it, and calibration is domain-specific.
TypeSafe's own guidance is to set thresholds conservatively and tune them on your data
([[typesafe-docs]]). A deployment with no labeled sample to check thresholds against has
routing logic but no evidence that it routes correctly.
