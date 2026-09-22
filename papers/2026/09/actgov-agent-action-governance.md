# ActGov: Governing LLM Agent Actions via Policy-Constrained Validation

**Authors:** Kaiyuan Zhang, Yuke Peng, Ke Jiang, Yinqian Zhang
**Institution:** Not stated in the arXiv metadata/listing
**Published:** September 21, 2026 (arXiv v1)
**Link:** https://arxiv.org/abs/2609.24446
**Code:** Not linked in the abstract

---

## Problem Statement

LLM agents increasingly run long-horizon workflows by chaining external tool calls. That means untrusted outputs (a fetched webpage, a file, a tool response) can influence the *next* action, and the agent can drift past what the user actually authorized. Existing defenses fall into two camps, and both are brittle:

- **Isolate/deny injected content** — works on known injection shapes, fails on novel ones, costs utility.
- **Constrain execution with predefined plans and static policies** — breaks the moment a workflow branches dynamically, and scales poorly as the tool ecosystem grows.

The core gap: enforcement happens at the wrong granularity and at the wrong time. You want to check every proposed action *before* it has an external effect, and you want the policy to be derived rather than hand-written — but you also can't trust the LLM to correctly flag malicious instructions.

## Methodology (technical)

ActGov is a runtime enforcement framework with a pre-runtime and a runtime half, built on a "unified semantic model" of four things: **authorization, actions, runtime context, and security constraints**.

**ActGov-Policy (offline / iterative construction):**
1. Synthesizes an initial policy set from *tool specifications*, *benign task examples*, and *observed failure traces*.
2. Each policy update is checked with **SMT-based counterexample verification** — i.e. an update is only accepted if no counterexample exists that would let a disallowed action through (or block a legitimate one). This gives a formal-ish guarantee over the policy set rather than relying on the author's intuition.

**ActGov-Runtime (online / per-action enforcement):**
1. Abstracts each tool call into **finite policy records**.
2. Permits the call **only if** it stays within the **task-scoped authorization boundary** *and* satisfies all applicable policies.
3. This is per-action, so authorization is preserved across long-horizon, dynamically branching workflows — the property static-plan defenses lose.

The critical design choice is that enforcement is **model-independent**: the guardrail does not ask the LLM whether an instruction is malicious. It checks whether the *action* is authorized given context. That sidesteps the entire "the model can be talked out of its judgment" attack class.

## Key Results (with numbers)

- Evaluated on **AgentDojo** and **AgentDyn** benchmarks, across multiple models and attack configurations.
- **Consistently reduces the success rate of indirect prompt-injection attacks** while **preserving task utility**.
- **Significantly outperforms existing defenses** on both axes.
- Authors' framing: demonstrates fine-grained authorization over dynamic agent executions "without relying on the underlying LLM to correctly identify malicious instructions."

(Exact percentage tables are not in the abstract; the paper reports the reductions per benchmark/model/attack configuration.)

## What's Novel

1. **SMT-verified policy synthesis** — the policy set is *grown* from tool specs, benign tasks and failure traces, but every update is machine-checked for counterexamples. Most agent-governance work hand-writes rules or learns them without formal checking.
2. **Per-action runtime enforcement with a task-scoped authorization boundary** — the boundary moves with the task, which is what makes it survive dynamic branching.
3. **Model-independence** — the defense does not depend on LLM classification of intent, which is the dominant failure mode of prompt-injection classifiers.
4. **Unified semantic model** bridging authorization + action + context + constraints, so the same policy language covers both the offline build and the runtime check.

## My Connection (to Manny's work)

- This is the *defensive* counterpart to your offensive agent work. If enterprises adopt tool-call-level authorization (and they will — it's the natural MCP-gateway design), then the interesting red-team question shifts from "can I inject an instruction?" to **"can I construct a sequence of individually-authorized actions that achieves the objective?"** Start building attack chains that each stay inside the task-scoped boundary — those are the ones ActGov's evaluation likely under-covers.
- The **offline policy-construction phase is itself a target**: policies are built from *observed failure traces*. If an attacker can shape what failure traces the operator collects, they shape the policy set. Worth probing in any deployment that uses trace-driven policy synthesis.
- Directly reusable in a report: it gives you a precise vocabulary ("task-scoped authorization boundary," "finite policy records") for describing why a given MCP deployment's guardrail fails.
- AgentDojo/AgentDyn are now the standard eval pair — worth standing up so your findings are directly comparable to the defense literature.

## What I Learned (plain English)

The right place to stop a hijacked agent is not the model's judgment — it's the tool call. ActGov works by formalizing, before anything runs, what the agent is allowed to touch for *this* task, then checking each action against that boundary as it happens. Because the check is about authorization rather than intent, it doesn't matter how cleverly the injected instruction is worded. The clever part is that the rules aren't written by hand: they're generated from the tool definitions and from past failures, and every addition is proven by searching for a counterexample. That makes the guardrail scale with the tool ecosystem instead of rotting as soon as a new plugin appears. The takeaway for a red teamer: once a target adopts this, your exploit isn't a prompt — it's an action sequence that looks legitimate one step at a time.
