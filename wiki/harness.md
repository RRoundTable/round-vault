---
type: page
tags: [harness, agent-workflow, tool-use, memory, agent-runtime]
category: agent-architectures
summary: "The system around a base model that orchestrates planning, tools, context, artifacts and evaluation — and the layer that improves before the weights do."
---

# Harness

The **harness** is the system surrounding a base model that orchestrates execution: how
the model thinks and plans, calls tools and acts, perceives and manages context, stores
artifacts, and evaluates results ([[lilian-weng-harness-engineering]]). It is the layer
between raw intelligence and the real world, and the claim that makes it interesting is
that this layer contributes about as much to deployed capability as the model's own
post-pretraining evals do — Claude Code and Codex being the demonstration.

The word marks a shift in what agent engineering *is*. The 2023 formulation was
"agent = LLM + memory + tools + planning + action". A harness adds workflow design,
evaluation, permission control, and persistent state management on top of that — which
moves the subject out of prompt templating and into runtime and software-system design.

## The OS analogy, and what it buys

Weng's framing is that a harness should behave like an operating system: encapsulate
complicated logic behind a simple interface, stay deliberately generic rather than
accumulating task-specific rules, and lean on existing software-engineering practice so
that the model's pretraining knowledge transfers. The prediction that follows is
standardization — configs, tool interfaces, and protocols converging across the industry
the way syscalls did.

The analogy also sets up the strongest objection to self-modifying harnesses. If a
program may edit the OS it runs on, abstraction boundaries break; the editable surface has
to be designed rather than assumed. See [[harness-reward-hacking]].

## Three design patterns

**Workflow automation.** A goal-oriented loop — plan, execute, observe/test, improve,
execute again — that runs *until* the goal is met, with proactive requests to the user
when the specification is ambiguous. The part that distinguishes a harness from a prompt
chain is that the model analyzes its own trajectories and failure cases and iterates
through an **agent runtime**, not a static template. Karpathy's `autoresearch` repo is
cited as a clean instance.

**File system as persistent memory.** In long-horizon rollouts the artifacts — experiment
logs, code diffs, paper summaries, error traces, past trajectories — grow far past the
context length the model was trained for. The harness should not carry the workflow in
context; it should keep durable state in files. Two consequences follow. Reading, writing
and editing files via `bash` is a *foundation* skill that every model improvement lifts,
so filesystem-backed memory rides the capability curve for free. And durable state is what
makes recovery after interruption possible at all.

**Sub-agents and backend jobs.** Spawn subagents to search multiple hypotheses or run
experiments concurrently, delegating isolated subtasks without polluting the main context.
The parent then needs a small process manager: launch, inspect logs, cancel failures,
merge results. The design rule is that **parallelism must be explicit and inspectable** —
subagent output that lives only in a transient chat context goes stale and invisible,
while output stored as files, logs and status records lets the parent reason over its own
execution history.

Patterns two and three share one idea: *write it down where the next reader can find it*.
A file survives a context window, a crash, and a session boundary; a message in a transcript
survives none of them.

## The coding-agent interface has stabilized

The core tool surface has converged across Claude Code, Codex, OpenCode and Cursor-style
agents:

| Group | Tools |
| --- | --- |
| File system | discovery, read, write, edit, patch |
| Shell | bash, PowerShell |
| IO | LSP, git |
| External context | MCP, skills |
| Web | search, fetch |
| Artifacts | documents, images |
| Backend processes | job management |
| Agent delegation | spawn, resume |

The convergence is itself the evidence for the OS analogy: independent teams arriving at
the same syscall table suggests the abstraction is real rather than a vendor's product
decision. It also matters for evolutionary methods — a stable, small tool surface is what
makes a harness a searchable object rather than a bespoke artifact
([[self-improving-harness]]).

MCE (Ye et al. 2026) makes the point sharply by running *both* levels of its optimization
inside a standard agentic coding environment with exactly this tool set —
`Read, Write, Edit, Bash, Glob, Grep, TodoWrite` — rather than any special-purpose
optimizer machinery.

## Decision calls inside the loop

The loop above spends a full model call on every step, including the many steps whose
output is really a small decision: which model should handle this, is this tool call safe,
is this done. LangChain's integration of [[jev]] moves those decisions to a typed decision
model exposed as **middleware**. A router picks the model for a run, and a gate checks
tool calls before they execute, while the LLM keeps the open-ended work
([[langchain-jev-harness]]). Because the decision model accepts the agent's own message
history as its state, it can be dropped in at any hook without reformatting context.

This fits the OS analogy better than it first looks. A harness already decides things
with deterministic code (permissions, budgets, retries). A decision model adds the
decisions that are too fuzzy for code but too frequent and too narrow to justify an LLM
call. The pattern that makes them safe is [[confidence-gated-routing]]: act on confident
answers, escalate the rest. The first public integration doesn't implement that
escalation yet.

## Harness layer versus core intelligence

Weng's near-term prediction, stated as a prediction and not a result:

- Harness engineering moves toward **meta-methodology** — improving the machinery that
  produces good answers rather than the answers — with fewer heuristic rules and more
  general mechanisms. The harness becomes an optimization target.
- Mature harnesses then enable auto-research loops for model self-improvement, while
  smarter models keep harnesses from over-engineering.
- Eventually many harness improvements are **internalized** into model behavior, but the
  interface to external context and tools remains.

The precedent she offers is prompt engineering. Manual prompt tricks became less central
as instruction tuning and reasoning improved — but *the need to specify goals,
constraints, context, and evaluation did not disappear*. The expectation is the same
shape here: the tricks get absorbed, the interface does not.

Internalization already has one measurement behind it. ThetaEvolve's complaint about
AlphaEvolve is that it is "a pure inference system that models cannot internalize the
evolving strategies"; adding test-time RL produces checkpoints that make **faster progress
on unseen tasks**, i.e. the search strategy moved into the weights
([[harness-survey-arxiv-abstracts]]). Hyperagents reports the same thing at the meta level —
persistent memory and performance tracking, learned once, transferring across domains and
accumulating across runs ([[evolutionary-program-search]]).

The cautionary evidence cuts against reading this too optimistically. STOP improved
downstream performance on GPT-4 and degraded it on weaker models; harness quality is
downstream of model quality, not a substitute for it. See [[self-improving-harness]].

## Outside coding, the harness barely exists

The convergence above is a coding-agent phenomenon. Continual Harness states the gap plainly:
coding harnesses like Claude Code and OpenHands wrap models with tools, memory and planning,
but **no equivalent exists for embodied agents' long-horizon, partially-observable
decision-making** ([[harness-survey-arxiv-abstracts]]).

Hyperagents explains why that asymmetry is structural rather than a matter of effort. Harness
self-improvement compounds in coding because *evaluating* and *self-modifying* are both coding
tasks, so gains in the task skill are gains in the improvement skill — an alignment that does
not hold anywhere else ([[evolutionary-program-search]]). Coding agents got good harnesses
first for the same reason they can improve them.
