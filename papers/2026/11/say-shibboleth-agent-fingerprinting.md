# Say Shibboleth: Fingerprinting the LLMs and Harnesses Behind Agentic Attacks

**Authors:** Dario Pasquini, Reworr, Giuseppe Ateniese, Manish Bhatt, Sergej Epp
**Institution:** Mixed academic/industry author team (affiliations not listed in the abstract; Sergej Epp is known for SAP security research)
**Published:** Submitted November 9, 2026 (arXiv:2611.12386), v0.1
**Link:** https://arxiv.org/abs/2611.12386
**Code:** Not stated in the abstract

---

## Problem Statement

When an autonomous agent attacks your infrastructure, the first question in the incident review is the one nobody can answer: **what was it?** Which model, running in which harness, with which guardrails? Attribution today is guesswork — you see tool calls in logs and infer "probably Claude" from the writing style of an error message.

This matters more than curiosity. Knowing the harness tells you what defenses the attacker has, what rate limits they're working under, how they hide secrets, and which of *your* controls will actually engage. Knowing the model tells you the capability ceiling of the operation and whether it's a script kiddie with a frontier API or a state program.

The paper asks whether attacker configuration can be **reliably inferred from telemetry alone.**

## Methodology

Two complementary inference channels, both designed to work *without* a successful prompt injection — which is the key methodological move, since a fingerprinting technique that requires you to already have won the attack is worthless for defenders.

**1. Shibboleth traps (declarative inference).** Modular honeypot services that gate access to a resource the agent wants behind an interaction that is necessarily informative. Classic "shibboleth" logic: to get past the gate, the agent must produce something that only a particular model would produce — model declarations in setup flows, characteristic writing in a required justification, and so on. Because the informative interaction is a *precondition for the resource*, the agent self-identifies as part of doing its job.

**2. Behavioral probes (harness inference).** General mechanisms that indirectly probe peculiar execution behaviors of a harness using only effects visible in *subsequent requests*:
- how the harness **masks secrets** in tool output (what shape does the redaction take, what survives)
- how it **enforces command time limits** (what happens at the boundary, what error text comes back)
- how it **schedules tool calls** (serialization, batching, concurrency fingerprints)

These are passive-ish: you observe consequences, not internals.

**Evaluation:** 15 candidate models for declarative + behavioral classification; behavioral classifiers tested on **held-out harnesses and held-out task families** (not just held-out samples); separate experiment across four harnesses with fixed probe rules. Plus an adversarial condition where the agent is instructed to conceal its identity.

## Key Results

- **90% average top-1 accuracy** distinguishing among **15 candidate models**, combining declarative and behavioral signals.
- **92% harness identification accuracy** across four harnesses using fixed probe rules.
- Behavioral classifiers generalize to **held-out harnesses and task families** — the result that separates this from overfit fingerprinting.
- **Concealment instruction degrades declarations but not behavior**: agents told to hide their identity still "leave useful behavioral evidence even as their declarations become uninformative."

## What's Novel

The honest framing of the contribution: not "we can identify models" (people have tried that), but **"the harness leaks independently of the model, and the leak survives an agent actively trying to hide."**

Two specific novelties:
1. **Gating resources behind informative interaction.** Instead of trying to elicit a fingerprint, you make the fingerprint the toll for passage. This is a honeypot design pattern that transfers to any agent-facing service.
2. **Harness-behavior probes that require no injection.** Secret-masking shape, timeout boundary error text, and tool-call scheduling are all observable from the outside and are *implementation-specific* in a way that model output is not.

The negative-ish result is arguably the most valuable: self-concealment instructions don't buy the attacker much, because behavior is harder to suppress than words.

## My Connection

This is the defensive mirror of everything else in this digest. Fable 5.2 and GPT-6 Cyber are being benchmarked on multi-stage operations; ShadowBench shows routers taking unauthorized paths; StealthyPoison shows skills being weaponized. Say Shibboleth asks the question that comes after: **when one of those operations hits your infrastructure, can you tell what hit you?**

For agent-security work specifically, the harness-fingerprinting channel is a new argument for a familiar conclusion — the harness is a security boundary in its own right, and it's *observable* by adversaries. If your harness's timeout error message, redaction pattern, and tool-call scheduling are distinctive, you are advertising your stack.

Practical use: deploy shibboleth traps as canaries in front of agent-reachable resources (API keys, "documentation" endpoints, fake admin panels). Each hit is both an alert and an attribution data point.

## What I Learned

Fingerprinting doesn't need the attacker to cooperate, because *tool use is behavior*, and behavior leaks implementation. The corollary for defenders is uncomfortable: any harness feature that makes you more usable (clear errors, helpful redaction messages, fast concurrent tool calls) also makes you more identifiable. Attribution and opsec are in direct tension, and most harnesses are optimized entirely for the first.
