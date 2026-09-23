---
type: source
tags: [harness, decision-models, routing, permissions, agent-workflow]
category: agent-architectures
summary: "LangChain's integration of Jev into agent harnesses as middleware: a per-run model router and a pre-execution tool-risk gate, and what the post leaves out (no confidence-based escalation, routing decided once per run)."
raw: raw/2026-09-23-langchain-jev-harness.md
url: https://www.langchain.com/blog/building-a-harness-with-jev
---

# LangChain: building a harness with Jev

LangChain's post on integrating [[jev]] is the first account of a decision model placed
*inside* an agent [[harness]] rather than used as a standalone classifier. Its thesis is
that tool calling and structured outputs made LLMs usable in software, but the agent loop
is still slow and costly because **every decision is another full model call**. A fast
typed decision model can take over those intermediate decisions while the LLM keeps
open-ended reasoning and generation. Jev is explicitly "not a drop-in replacement for an
LLM" but a complement to it. The post ships two middlewares. It is a vendor-integration
post with no measurements of its own. Its speed and cost figures are TypeSafe's, quoted.

## Integration surface

`langchain-typesafe` exposes Jev as `TypeSafeClassifier`. `.invoke()` takes a state and
questions and returns typed results rather than a chat message. The state can be plain
text, structured data, or **LangChain messages**, so a node or middleware hook can pass
the agent's existing context straight in with no reformatting. That is what makes the
decision model cheap to add at any point in the loop.

## Middleware 1: model routing

`ModelRouterMiddleware` takes named choices, each a model plus plain-language criteria
(the example routes "direct lookups, extraction, and localized changes" to a fast model
and "architecture and high-stakes decisions" to a capable one), with the instruction
"Choose the least costly model that can complete the task." The probabilities and
confidence stay available in agent state.

Two details matter more than the post lets on. The router **selects from the latest user
message and uses that model throughout the run**, so it is a per-request dispatch, not a
per-step one. A misroute persists for the whole run. And the example shows no confidence
threshold on the routing decision itself: the confidence is recorded but not used to
escalate.

## Middleware 2: tool-risk gating ("Auto Mode")

`AutoModeMiddleware(tools=["bash"])` uses Jev to check each tool call for risky actions
and **block it before the tool executes**. The motivation is stated plainly: agents "can
receive bad instructions (either naturally or from a motivated enough attacker)". Coding
harnesses such as Claude Code, Codex and Cursor already ship classifiers for dangerous
actions, but "locked away in the closed source parts of the harness." The post's argument
is that a cheap, fast classifier makes the pattern available to every agent.

The post doesn't say what the gate sees. Classifying a call in context means reading
the same message history that might carry the injected instruction. That makes this a
probabilistic permission check, a different kind of control from a structural one. See
[[harness-reward-hacking]].

## What is missing

- **No confidence-gated escalation.** Neither middleware sends low-confidence decisions to
  a stronger model or a person. That is the pattern calibration exists to enable
  ([[confidence-gated-routing]]).
- **No numbers.** No routing accuracy, no false-block rate for the gate, no end-to-end cost
  change for an agent run.
- Community examples (browser-use agents, a trading agent, email triage at scale) are
  linked as tweets, not evaluated.
