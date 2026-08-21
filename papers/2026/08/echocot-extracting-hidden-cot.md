# EchoCoT: Extracting Hidden Chain-of-Thought from Large Reasoning Models

- **Paper:** https://arxiv.org/abs/2608.20055
- **Authors:** Yiting Qu, Ziqing Yang, Chi Cui, Ye Leng, Junjie Chu, Yang Zhang
- **Published:** 2026-08-20
- **Code:** Not listed in abstract

---

## Problem Statement (plain English)

Frontier labs treat their models' hidden chain-of-thought (CoT) as a valuable proprietary asset — they show summaries, not the full reasoning. Can a black-box attacker extract those hidden CoTs near-verbatim through nothing but API calls? If yes, the reasoning "crown jewels" of proprietary models are exfiltratable, and any safety reasoning hidden in CoT is exposed too.

## Methodology (technical)

- Identify a previously overlooked "reasoning replay surface" between tool calls — the model's internal reasoning gets echoed back in API-returned fidelity signals.
- EchoCoT: a multi-step attack that iteratively extracts hidden CoTs using those API-returned fidelity signals as a guide.
- An LLM-based optimization framework that automatically searches for a universal injection trajectory across datasets (no per-dataset hand-tuning).
- Evaluated on three open-source and five frontier proprietary LRMs.

## Key Results (with numbers)

- Open-source LRMs: up to 66.4% near-verbatim extraction success; extracted traces within 10% of target length with ≥90% token-exact match.
- The same injection trajectory generalizes to unseen datasets: up to 80% extraction success.
- Frontier proprietary models: substantial fraction of extracted CoTs closely align with provider-reported reasoning lengths and CoT summaries.
- Long CoT extraction: on Gemini-2.5, extracted 33,463 tokens from a 32,948-token target CoT.

## What's Novel

- First systematic black-box demonstration that hidden CoTs can be extracted near-verbatim, not just paraphrased.
- The tool-call "reasoning replay surface" is a new attack vector specific to agentic/reasoning APIs.
- Universal injection trajectory found automatically — the attack transfers across datasets without adaptation.

## My Connection (to Manny's work)

Two angles. Red team: reasoning models exposed via agentic APIs (tool-calling loops) are leaking their hidden CoT — that's a measurable IP-theft and safety-reasoning-exposure risk for any client running proprietary reasoning models. Defensive: if hidden CoT is extractable, "we don't expose our reasoning" is not a control — CoT summaries and reasoning lengths are already a fidelity oracle; plan monitoring for repeated tool-call replay patterns that look like extraction loops.

## What I Learned (plain English)

- Hidden reasoning is not hidden: tool-call boundaries leak fidelity signals that make extraction tractable.
- One injection trajectory generalizes across datasets — attacks here are cheap to scale.
- Reasoning length/summary disclosures are themselves information; treat them as an oracle an attacker can exploit.
