---
type: source
tags: [calibration, decision-models, routing, classification, bibliography]
category: machine-native-models
summary: "Abstracts of 17 papers bearing on Jev's claims: which ones support TypeSafe's premise that preference and verifiable-reward training break calibration, the published analogues of RLCD, and the ones that undercut 'can't hallucinate' and 'new category'."
raw: raw/2026-09-23-jev-related-arxiv-abstracts.md
url: https://arxiv.org/
---

# arXiv abstracts related to Jev

TypeSafe cites no papers, so this set was chosen by the wiki, not by TypeSafe: 17
published works that bear on [[jev]]'s claims, fetched from the arXiv API on 2026-09-23.
All IDs resolved. The short version is that the literature **supports the premise and
contests the novelty**. Several independent papers show that RLHF and binary-reward RL
degrade calibration, and several already train calibration with proper-scoring-rule
rewards, which is the closest published form of what "RLCD" is described as. At the same
time, hallucination theory treats a wrong yes/no answer as the elemental hallucination,
and zero-shot classification over runtime-specified labels dates to 2019.

**Scope limit: abstracts and metadata only.** Headline results and stated boundary
conditions are reliable at this level. Method detail is not.

## The premise holds: standard post-training breaks calibration

- **Pretraining is not the problem.** Larger models "are well-calibrated on diverse
  multiple choice and true/false questions when they are provided in the right format"
  (Kadavath et al. 2022, arXiv:2207.05221). Those are exactly Jev's Choice and Noul
  shapes. The same paper trains P(IK), a model's probability that it knows the answer,
  which partially generalizes but "struggle[s] with calibration of P(IK) on new tasks."
- **RLHF damages logit calibration.** For ChatGPT, GPT-4 and Claude, *verbalized*
  confidences beat the model's own conditional probabilities, often cutting expected
  calibration error by a relative 50% (Tian et al. 2023, arXiv:2305.14975). The
  probabilities RLHF leaves behind are the wrong thing to threshold on.
- **Binary-reward RL damages it too.** Correctness-only rewards "do not penalize guessing
  or low-confidence outputs" and degrade calibration (Damani et al. 2025,
  arXiv:2507.16806). RLVR makes models "excessively over-confident in incorrect answers"
  (Ma et al. 2026, arXiv:2603.09117). Binary rewards make models "good test-takers rather
  than honest communicators" (Wu et al. 2025, arXiv:2512.19920).
- **Modern networks were miscalibrated before LLMs.** Depth, width, weight decay and
  batch norm all hurt calibration, and **temperature scaling**, a one-parameter post-hoc
  fix, is "surprisingly effective" (Guo et al. 2017, arXiv:1706.04599). Any claim of
  trained-in calibration should be compared against this cheap baseline.

## Published analogues of RLCD

| Paper | Reward | Key result |
| --- | --- | --- |
| **RLCR**, Damani et al. 2025 (2507.16806) | correctness + **Brier score** | proves any bounded proper scoring rule reward yields accurate *and* calibrated models; improves calibration with no accuracy loss, in and out of domain; beats post-hoc confidence classifiers |
| **Rewarding Doubt**, Bani-Harouni et al. 2025 (2503.02623) | **log scoring rule** on stated confidence | optimal policy is perfectly calibrated; generalizes to unseen tasks without further tuning |
| **DCPO**, Ma et al. 2026 (2603.09117) | calibration *decoupled* from the reasoning objective | shows a "fundamental gradient conflict" between accuracy and calibration; keeps GRPO accuracy with the best calibration |
| **Behaviorally calibrated RL**, Wu et al. 2025 (2512.19920) | strictly proper scoring rules; model may abstain or flag claims | a 4B model matches frontier calibration on SimpleQA despite much lower accuracy: calibration is "a transferable meta-skill decouplable from raw predictive accuracy" |

DCPO is the strongest support for TypeSafe's position. If accuracy and calibration
gradients conflict, calibration needs its own objective rather than a term added to
someone else's. Behaviorally calibrated RL is the strongest support for a small,
specialized calibrated model being possible at all.

**Name collision:** RLCD already names *Reinforcement Learning from Contrastive
Distillation* (Yang et al. 2023, arXiv:2307.12950), a preference-pair alignment method
unrelated to calibration.

## Against "can't hallucinate"

- Hallucinations "originate simply as errors in binary classification", and persist
  because evaluations reward guessing over admitting uncertainty (Kalai et al. 2025,
  arXiv:2509.04664). On this account, a confidently wrong Noul is not an alternative to
  hallucination. It is hallucination in its elemental form.
- A *calibrated* generative model must hallucinate "arbitrary" facts at roughly the rate
  of facts seen once in training, even with perfect data. But there is "no statistical
  reason" for hallucinating systematic facts, and "different architectures and learning
  algorithms may mitigate" those (Kalai & Vempala 2023, arXiv:2311.14648). This leaves
  room for a non-generative design to do better, on the systematic kind only.

## Against "new category"

- **Zero-shot classification over arbitrary, runtime-specified labels** was benchmarked
  and unified as textual entailment in 2019 (Yin et al., arXiv:1909.00161). That is the
  core capability Jev exposes as Choice and Noul.
- Encoder-only models are "the workhorse of numerous production pipelines" for
  classification, and ModernBERT is a major Pareto improvement with 8,192-token context
  (Warner et al. 2024, arXiv:2412.13663).

What neither covers is the combination Jev claims: task specified in natural language at
call time, frontier-level judgment, and trained calibration.

## Routing and abstention prior art

- **Selective classification** lets the user set a target risk level and rejects inputs
  to meet it with high probability, e.g. 2% top-5 ImageNet error at 99.9% probability
  with ~60% coverage (Geifman & El-Yaniv 2017, arXiv:1705.08500).
- **FrugalGPT** cascades LLMs, matching GPT-4 at up to 98% lower cost (Chen et al. 2023,
  arXiv:2305.05176). **RouteLLM** trains strong/weak routers on preference data, cutting
  cost by more than 2× with routers that still work when the underlying models are
  swapped (Ong et al. 2024, arXiv:2406.18665).

## Why decomposition works, and where System 1 ends

- Format restrictions degrade LLM reasoning, and stricter formats degrade it more (Tam
  et al. 2024, arXiv:2408.02442). This explains why a model that emits no reasoning
  tokens needs questions small enough to require none.
- System-2 techniques can be distilled into System-1 generation, except "complex math
  reasoning tasks requiring chain-of-thought" (Yu et al. 2024, arXiv:2407.06023). That is
  a published boundary for what a System One model can be expected to absorb.
