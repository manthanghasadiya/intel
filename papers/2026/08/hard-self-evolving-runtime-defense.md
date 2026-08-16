# Beyond Handcrafted Security: Towards Self-Evolving Defense for LLM Agents (HARD)

- **Link:** https://arxiv.org/abs/2608.12977
- **Authors:** Jiajun Ruan (U. Minnesota), Peiyang Li (Ant Group), Yukun Chen (Tsinghua U.), Fengting Li, Chao Feng (Zhejiang U.)
- **Published:** Aug 13, 2026 (v1) — announced in the Fri Aug 14 arxiv batch
- **Categories:** cs.CR
- **Code:** not listed on abstract page

## Problem Statement (plain English)

Runtime defenses for LLM agents (filters, guardrails, tool gates) are handcrafted and static. Attackers adapt — new prompt injections, memory poisoning, skill poisoning — and every new failure mode requires a human to manually diagnose the failure and edit the defense. That doesn't scale. HARD asks: can the agent harness itself *autonomously evolve its own defenses* from observed failures, the way an immune system does?

## Methodology (technical)

- **Harness-centric formulation:** runtime defense is reframed as optimization over the executable harness H = (φ_H, ψ_H), where φ_H governs *context construction* (what the model sees) and ψ_H governs *action interpretation* (what tools the agent may call). This unifies existing defenses (input filtering, guard models, least-privilege, execution isolation) as interventions on one of these two interfaces.
- **Failure-driven evolution loop:** failed trajectories are collected; an LLM-based *trace router* attributes each failure to the responsible defense artifact; artifact-specific LLM *evolvers* (policy evolver for context, gate evolver for actions) refine only that artifact, with prompts explicitly discouraging benchmark memorization and demanding generalizable security invariants.
- **Design:** each attack category evolves independently; train/test splits are deterministic and shared; failures are batched per round to cut LLM calls; gate predictions with confidence < 0.55 are reassigned to the policy evolver.
- **Setup:** execution agent fixed at DeepSeek-V4-Flash; safety/utility judges at GLM-5; router+evolvers at GLM-5.2 (main), with backbone ablation across Claude Opus-4.6, Qwen3.7-Max, GPT-5.5.
- **Benchmark:** AgentCanary, covering direct prompt injection (DPI), indirect prompt injection (IPI), memory contamination (MC), skill poisoning (SP), plus two adaptive settings — dynamic attack evolution (DAE) and long-horizon progressive attacks (LPA).

## Key Results (with numbers)

- **Static attacks — HARD-Both ASR:** 15.4% (DPI), 1.0% (IPI), 6.7% (MC), 10.2% (SP), vs 13–66% for handcrafted defenses; benign utility held at 91.9–95.0%. Significance: paired McNemar p < 10⁻⁸ vs every baseline.
- **Utility under attack:** UA rose from 56% → 86% (memory poisoning) and 52% → 92% (skill poisoning).
- **Adaptive attacks:** DAE — HARD-Both 26.5% ASR vs 30.1% best handcrafted; LPA — HARD-Policy 4.8% and HARD-Both 12.1% vs 24.1% best handcrafted.
- **Backbone ablation (memory contamination):** no evolution 63.9 ASR / 57.8 UA → GLM-5.2: 13.9/85.9; Claude Opus-4.6: 7.7/85.3; Qwen3.7-Max: 13.5/74.1; GPT-5.5: 20.3/7.2 (over-restrictive — shows evolution quality varies by backbone).
- **Routing matters:** HARD-Gate grew 0 → 97 execution rules over 5 rounds (ASR 36.6% → 23.2%); HARD-Both beat single-artifact evolution on static attacks, but HARD-Policy won on LPA — gate rules are brittle against reworded/obfuscated commands, policies transfer better.

## What's Novel

First principled framework to treat runtime defense as harness optimization with *autonomous evolution* — defenses repair themselves from failure traces without retraining the model or manual security engineering. Provides a unified vocabulary (context construction vs action interpretation) for the whole runtime-defense literature.

## My Connection (to Manny's work)

HARD is a harness-level security primitive, not a model-level one — exactly the layer Manny operates in. The trace-router/evolver loop is a blueprint for autonomous red-team defense tuning in agent harnesses, and the finding that gate rules (content-matching) are evadable while policy invariants transfer is directly actionable for designing MCP tool gates and sandbox policies. The LPA result (4.8% ASR) also validates long-horizon progressive-attack testing as a high-value eval design.

## What I Learned (plain English)

- Where a defense sits in the harness (what the agent sees vs what it's allowed to do) determines how it fails — and both must evolve together for static attacks, while policy-level learning wins against adaptive ones.
- Content-matching rules are the weak link: obfuscation defeats them. Semantic invariants generalize.
- An "evolver" LLM can turn a handful of failure traces into reusable security principles — and a single observed failure can be enough to synthesize an effective defense.
- Not all evolution backbones are equal: stronger models evolve better defenses, but can also over-restrict and destroy utility under attack.
