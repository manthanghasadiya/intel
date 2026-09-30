# CounterSteer: Suppressing Indirect Prompt Injection with Activation Steering

**Authors:** Mark Russinovich
**Institution:** Microsoft (author is Microsoft's CTO/Technical Fellow — notable industry authorship)
**Published:** 2026-09-29
**Link:** https://arxiv.org/abs/2609.36570
**Code:** Not stated in abstract

---

## Problem Statement (plain English)

Indirect prompt injection makes an agent treat untrusted retrieved text (a web page, a file, a tool result) as if it were an instruction. Most defenses try to **detect** the bad content — but detection is something an attacker can learn to evade. CounterSteer instead asks: can we suppress the *behavior* of following embedded instructions directly inside the model, at inference time, without retraining?

## Methodology (technical)

CounterSteer is an **inference-time activation-steering defense** with a per-model, five-step recipe:

1. Build **paired episodes** that differ *only* in whether an embedded instruction is followed.
2. Fit a **residual-stream direction** that separates the "follows injected instruction" cases from the "ignores" cases.
3. Retain that direction **only if it passes pre-specified causal and capability gates** (i.e., it must both change behavior and not break tasks).
4. At deployment, **subtract the direction from every tool-result token during prefill**.
5. The edit is **always on** — there is no detection decision for the attacker to evade.

It requires **no fine-tuning, no auxiliary model, and no added tokens** — only white-box serving access and knowledge of the tool-result span boundaries.

## Key Results (with numbers)

- **Held-out attack success:** falls from **0.21–1.00** (undefended) to **0.00–0.17** (defended) across **five open-weights models (8B–106B, five vendor lineages)**.
- **AgentDojo compromise rate:** from **0.10–0.49** down to **0.006–0.079**.
- **Benign utility:** **93–100%** of typography-normalized benign utility retained (with larger task-dependent costs when reasoning over steered content).
- **Adaptive attacker:** a benchmark-level adaptive attacker reaching **0.67–0.73** undefended is held to roughly **a quarter** of that on the two most deeply evaluated models.
- **Comparison:** among defenses measured on capable models, those achieving lower compromise rates either **lost 22–89% of benign utility** or **fine-tuned the served weights**.
- **Robustness:** white-box gradient attacks through the deployed vector compromised **at most 2 of 52 episodes**; **none of 2,052 replayed human red-team attacks** succeeded.
- **Weakness:** **parameter manipulation** (attacker-chosen arguments in otherwise legitimate calls) is only **partially resisted — 13 of 18** succeeded. The decision becomes linearly readable at argument emission but is not removed by prefill- or decode-time steering, motivating **argument-provenance controls**.

## What's Novel

- **Always-on steering** removes the detect-then-block decision point — and therefore the evasion game — entirely.
- Fitting the direction from **minimal paired episodes** with **causal + capability gates** is a clean, reusable recipe for turning a behavioral failure into a single steerable direction.
- Honest scoping: the authors explicitly show which attack class (parameter manipulation) the method *doesn't* solve, which is unusually candid and directly actionable.

## My Connection (to Manny's work)

This is a rare defense that survives adaptive attackers and human red-teaming at scale — the numbers Manny asks for before taking a defense seriously. It also cross-references today's other paper: CounterSteer neutralizes *instructional takeover* but leaves *parameter manipulation* open, which is exactly the gap **ToolFence** targets. Together they sketch the real architecture: steer the model away from following injected text, **and** authorize effects with provenance for the arguments.

## What I Learned (plain English)

You don't always have to detect an attack to stop it. If a specific failure has a consistent internal "direction," you can subtract that direction from the model's activations whenever it reads untrusted text — always on, so there's nothing to evade. But it only fixes *instructions being followed*, not *legitimate tools being called with malicious arguments*; for that you still need argument-level provenance controls.
