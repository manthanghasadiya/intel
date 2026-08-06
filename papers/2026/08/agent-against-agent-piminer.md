# Agent Against Agent: An Agentic System for Automatic Prompt Injection Red Teaming (PIMiner)

- **Link:** https://arxiv.org/abs/2608.05108
- **Authors:** Yanting Wang, Chenlong Yin, Runpeng Geng, Jinyuan Jia
- **Institution:** Not stated on arXiv abstract page
- **Published:** 2026-08-05
- **Code:** Available (per abstract page comment)

---

## Problem Statement (plain English)

Prompt injection is the #1 practical threat to LLM agents, but red-teaming it well is slow and expensive. Existing SOTA red-teamers train an attacker model with reinforcement learning per target LLM — which means when a new model ships, you retrain from scratch, and the resulting attacker generalizes poorly to models it never saw. We need a red-teamer that can attack a *brand-new* target model immediately, with almost no queries, and still land injections.

## Methodology (technical)

PIMiner is an agentic (LLM-driven, tool-using) red-teaming system with two phases:

- **Training:** PIMiner is trained across a sequence of (dataset, target model) pairs. From this it builds a *strategy library* from scratch — a structured collection of injection tactics, templates, and contextual framing that worked across targets. The library, not model weights, is the transferable artifact.
- **Test time:** Given an unseen target agent, PIMiner retrieves/selects strategies from the library and adapts them on the fly, requiring only ~10 queries to the target per test sample. No additional training, no RL rollout against the new model.

The agentic loop (generate candidate injection → observe target response → refine strategy) is what lets it adapt cheaply per target instead of pretraining an attacker network.

## Key Results (with numbers)

Attack Success Rate (ASR) on two benchmarks:

| Target | IPIArena | AgentDojo |
|---|---|---|
| Gemini-2.5-Pro | 76.2% | 86.7% |
| GPT-5.1 | 61.9% | 53.3% |
| Claude-Sonnet-4.5 | 42.9% | 40.0% |

Key claims: strong zero-shot transfer to unseen targets, ~10 queries per test sample, outperforms RL-based attackers on generalization (the failure mode the paper targets).

## What's Novel

- Shifts the transferable artifact from *model weights* (RL attackers) to a *strategy library* — attack knowledge as a first-class, reusable asset.
- Agentic adaptation at test time means the red-teamer is cheap to deploy against every new model a lab ships.
- Directly addresses the practical pain point: red-teaming throughput for fast-moving model release cycles.

## My Connection (to Manny's work)

This is a buildable, weaponizable template for Intrud3r's own tooling: a strategy-library prompt-injection agent that can be pointed at any agent under test (MCP servers, coding agents, internal harnesses) without per-target training. The ~10-query budget makes it viable as a CI-gated red-team check. The obvious extension: apply the same library-transfer idea to tool-dispatch attacks (CoreBreak class) rather than pure text injection.

## What I Learned (plain English)

- RL-trained injection attackers don't transfer; knowledge-transfer beats weight-transfer for red-teaming.
- A small number of well-adapted queries beats brute-force fuzzing — strategy selection matters more than query volume.
- Claude-Sonnet-4.5 was the most resistant of the three tested targets — worth noting when choosing which models to run sensitive agent workloads on.
