# ECLIPSE: Self-Evolving Stealthy Prompt Injection Attack against Long-Horizon Agentic Systems

- **Paper:** https://arxiv.org/abs/2608.30441
- **Authors:** Shiqian Zhao, Yangfan Zhou, Xinfeng Li, Runyi Hu, Yechao Zhang, Yi Xie, Tianwei Zhang, Luu Anh Tuan
- **Published:** 2026-08-31
- **Code:** Not specified in abstract

## Problem Statement (plain English)

LLM agents (Codex, Claude Code, OpenClaw) now plan and execute long, multi-step tasks through repeated tool calls. That long horizon is a gift to attackers: prompt injection today is either a single loud instruction (easy for safety filters to spot) or intent smeared across stages (unreliable to execute). Nobody had a way to inject a *stealthy, verifiable plan* into a long-horizon agent. ECLIPSE is that way.

## Methodology (technical)

Two components:

1. **Stealthy Attack Trajectory Synthesis** — runs candidate tool chains in a sandbox, iteratively verifies which chain actually achieves the malicious objective, then renders the verified chain as a natural one-shot prompt (the direct injection).
2. **Tool-Chain Steering** — transfers the plan into the target environment via:
   - **Static Workflow Encoding (SWE):** embeds state-transition cues in the descriptions of target tools (indirect, tool-side injection).
   - **Dynamic Trajectory Correction (DTC):** supplies corrective signals when the victim's execution deviates from the planned chain.

Also introduces **LASE-Bench**: a long-horizon agent-safety benchmark with 120 malicious tasks and 198 unique tools; 96.7% of tasks require ≥5 tool calls.

## Key Results (with numbers)

- **96.7% attack success rate** on long-horizon agentic systems with no defense.
- **69.2% ASR** even under a common safety filter — beating the strongest baseline.
- Benchmark: 120 malicious tasks, 198 tools, 96.7% of tasks ≥5 tool calls.

## What's Novel

First framework to make long-horizon injection *self-evolving and stealthy*: the attack verifies its own tool chains before deploying them (sandbox), then maintains them at runtime (SWE + DTC) instead of hoping one static prompt survives a 20-step trajectory. Ships the first long-horizon agent-safety benchmark built for this regime.

## My Connection (to Manny's work)

This is the exact class of attack Manny's agent harnesses and eval pipelines will face: a prompt that looks innocuous, verified tool chains, mid-run correction. LASE-Bench is a ready-made eval suite to drop into red-team CI. If ECLIPSE-style attacks hit 69.2% past filters, our eval harnesses need them as a standard test vector — not an exotic one.

## What I Learned (plain English)

Long-horizon agents are *more* injectable, not less: the attack surface is the tool descriptions and the trajectory, not just the initial prompt. The defense implication is that static filters are near-useless; you need runtime monitoring of tool-call intent, provenance, and deviation from user intent — the same conclusion Pluto's Apify research reached from the product side.
