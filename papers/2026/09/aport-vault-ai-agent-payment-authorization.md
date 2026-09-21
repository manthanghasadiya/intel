# APort Vault: Benchmarking AI Agent Payment Authorization with the Open Agent Passport

**Authors:** Uchi Uchibeke (APort)
**Institution:** APort / Open Agent Passport project
**Date:** Submitted Sep 18, 2026 · cs.CR
**Link:** https://arxiv.org/abs/2609.22076
**Dataset:** https://huggingface.co/datasets/aporthq/vault-benchmark-v1 (225,964 evaluations, level passports, scoring code, analysis script)

---

## Problem Statement (plain English)

AI agents are starting to move real money. The central safety question isn't "can the model be tricked into wanting to pay the wrong person" — it's "even if it wants to, is it *able* to?" A benchmark that only measures the model's intent (did it ask to pay?) tells you nothing about whether the transfer actually clears. This paper builds a benchmark that separates the model's behavior from a hard, deterministic authorization boundary, and measures both across thousands of attacks.

The framing is unusually honest for an agent benchmark: the author argues that collapsing the many events in a payment attempt into one number is *how you get a benchmark that doesn't survive review*, so they report **five distinct events per evaluation** instead.

## Methodology (technical)

- Replays **4,371 human-written attacks** captured against a **live payment agent** during a public capture-the-flag event.
- **14 models from 8 labs**, five policy configurations, two replay tracks (with/without a deterministic **pre-action check** implementing the **Open Agent Passport (OAP)** specification).
- **225,964 evaluations** completed.
- Four difficulty **Levels** (1–4), where higher levels tighten the legitimate-recipient allowlist and raise the stakes; each level ships a "passport" (the policy).
- Five events reported per evaluation to avoid collapsing signal (e.g. request, authorization outcome, execution outcome).

## Key Results (with numbers)

- **Request rates are common and vary more by configuration than by model**, though each attack exists at exactly one configuration (so policy and attack cohort vary together):
  - **10.9%** of model-alone evaluations requested a payment at **Level 1**
  - **3.0%** at Level 2
  - **0.1%** at Level 3
  - **79.4%** at Level 4
- On the **1,293 Level 4 prompts**, each evaluated on every model, request rates ran **71.2%–84.3%**, and **809 prompts (62.6%)** elicited a request from **all fourteen models** — each ending in a successful payment to that level's allowlisted recipient.
- **The authorization boundary is where conditions diverge.** At Levels 2–4:
  - Transfers to **forbidden recipients**: **140 of 76,842** with the model alone → **0 of 69,297** behind the OAP layer.
  - On **68,970 matched model/prompt/track triples**: **105 → 0**.
  - The zero spans **790 source sessions**, giving a **per-session upper bound of 0.38%**.
- It was **not** achieved by refusing payments: **25,370 payments executed** behind the layer, while policy denied **187 of 25,640** evaluated transfer calls — **148** of them for a forbidden recipient.

**Bottom line:** a deterministic pre-action authorization layer drove unauthorized-recipient transfers to zero while preserving the overwhelming majority of legitimate payments.

## What's Novel

- A **deterministic, out-of-model authorization boundary** (OAP) evaluated at scale, rather than relying on model-side refusal.
- **Five-event reporting** per evaluation — a methodological correction to how agent benchmarks collapse behavior into a single misleading number.
- **Public attack corpus + scoring + analysis** (Hugging Face), and an honest statement of the limit: the zero is bounded per-session (0.38%), not proven absolute.
- Separation of "did the agent *request*" from "did the money *move*" — the distinction that makes the result meaningful.

## My Connection (to Manny's work)

- **Agent-payment red-teaming:** this is the benchmark format to replicate for client engagements — model-alone vs. policy-mediated, with matched triples so the comparison is fair. The "matched triple" methodology is worth stealing outright.
- **Defense you can recommend:** the result is the strongest published evidence to date that a deterministic pre-action check beats model-side guardrails for financial actions — the same architecture as the earlier **Whisper/AP2 agent-payment** work I covered.
- **Reporting discipline:** the five-event framing is a direct lesson for how I present agent-risk findings so they survive review (don't report one scary percentage; report the request→authorize→execute chain).

## What I Learned (plain English)

You cannot fix payment-agent safety inside the model. The paper shows that when you let the model alone decide, 140 forbidden-recipient transfers slipped through out of ~77k; when you put a boring deterministic allowlist check in front of the transfer, that dropped to zero **without** blocking legitimate work (25,370 payments still went through). The deeper lesson is methodological: a benchmark that reports "the model requested a bad payment X% of the time" is measuring intent, not harm — and intent is not the thing that empties a bank account. Measure the boundary, not the model's mood.
