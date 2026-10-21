# Proof-Carrying Contracts: Commitments AI Agents Cannot Break

**Authors:** Carlo Ciliberto, Alessandro Rudi, Enrico Tassi
**Institution:** Not listed on the arXiv abstract page
**Published:** October 20, 2026 (v1) — 70 pages, 8 figures
**Link:** https://arxiv.org/abs/2610.28484
**Code:** Not linked in the abstract

---

## Problem Statement

When one agent acts on behalf of a person or another agent — buying a service, delegating work, exchanging data — the terms it must respect live in instructions, policies or agreements written in natural language. Text cannot make an agent comply. It may err, be redirected by what it reads, or simply pursue its own objective.

The standard fix is a permission policy: enumerate what the agent may do, and prove that every permitted action is safe. That is expensive and brittle — you must anticipate every action in advance, and the policy says nothing about *why* an action is acceptable in context. For an agent economy to work, two mutually distrusting parties need agreements they can rely on **without inspecting or trusting each other's agents**.

## Methodology

The paper introduces **Proof-Carrying Contracts (PC2)**. The mechanism is an inversion of the usual permission model:

1. **Parties specify the constraints to preserve** — invariants over system state that must hold throughout an interaction, not a list of allowed operations.
2. **Before any requested operation runs, the agent must supply a machine-checkable proof** that the operation preserves those constraints *in every case allowed by its model* of the system.
3. **An execution system called the Airlock checks the proof** and gates access to the operation. No proof, no execution.

The agent keeps full freedom over strategy — it chooses how to act — but it must justify each decision to a checker that does not trust it. This sidesteps the "design a policy and prove every permitted decision is safe" problem: the agent supplies the reasoning, the checker verifies it.

**Reusable capability modules** provide tools together with the guarantees they carry, so contracts can be composed out of pre-proven parts rather than rebuilt for each integration. The system is built in the tradition of proof-carrying code, with the contract as the carrier and the Airlock as the reference monitor.

## Key Results

- **Soundness theorem:** the agreed constraints hold at every instant during execution, **even for an adversarial agent** — the guarantee does not depend on the agent being honest or on the model behaving as prompted.
- **World-safety characterization:** the authors identify the conditions under which the formal invariant implies safety in the real world (i.e., when the model of the system is faithful), rather than claiming the two are equivalent.
- **Composition:** capability modules let contracts be assembled and reused, with guarantees inherited from the modules.
- **No empirical benchmark.** The contribution is formal; there is no agent task suite, no attack-success-rate evaluation, and no implementation performance numbers in the abstract.

## What's Novel

Most agent-safety work tries to constrain *behavior* — better prompts, guard models, classifiers over actions. PC2 constrains *admissibility* and shifts the burden of proof onto the party requesting the action. Two properties stand out:

- **Zero trust in the agent is a theorem, not a configuration.** Adversarial-agent soundness is proven, not assumed.
- **Constraints are specified by the parties, not the platform.** Neither side has to reveal its internals or trust the other's harness — which is the actual blocker for agent-to-agent commerce.

The honest limitation: this requires the agent to produce formal proofs, which means a model of the system and a proof language. That is a very different engineering world from an LLM emitting a tool call.

## My Connection

Manny's work sits on MCP servers and agent harnesses where the current enforcement primitive is a scope on a token plus a human approval prompt. PC2 is the theoretical ceiling for that design space: instead of "did the agent have `files:write`?", the question becomes "did the agent prove this write preserves the tenant-isolation invariant?"

The practical near-term takeaway is architectural, not literal: **make the enforcement point check a property of the operation rather than the identity of the caller**, and keep the enforcement outside the model's context. That is the same reasoning as kernel-level egress control and the Airlock metaphor is worth stealing for a blog post on why approval prompts are not a security boundary.

## What I Learned

"If the agent cannot break the rules" and "the agent chooses not to break the rules" are different claims, and only the first one survives an adversarial model. Proof-carrying contracts make the first claim checkable — and the price of that is that someone has to write down what "safe" means as a formal invariant. That specification work, not the proof checking, is where the real difficulty lies, and it is the same work most agent deployments skip entirely.
