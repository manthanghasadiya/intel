# SIR: Self-improving Red-teaming for Compute Use Agents

- **Paper:** https://arxiv.org/abs/2608.30207
- **Authors:** Chen Xiong, Zhiyuan He, Pin-Yu Chen, Stjepan Picek, Tsung-Yi Ho
- **Published:** 2026-08-31
- **Code:** Not specified in abstract

## Problem Statement (plain English)

Computer-use agents (CUAs) — vision-language models that see a screen and act on a real OS via mouse, keyboard, terminal — read untrusted content while working, so they're prime targets for indirect prompt injection (IPI). Existing CUA safety benchmarks use hand-written, fixed injections, which underestimate a *adaptive* adversary who learns from failures and improves. SIR is that adaptive adversary, automated.

## Methodology (technical)

Black-box IPI attack with two parts:

1. **Principle composition** — builds stealthy injections from a small library of reusable attack principles stated in plain language.
2. **Iterative feedback loop** — diagnoses the victim's *failed* trajectories, distills the bypasses into new named strategies, and re-applies them across tasks (self-improvement).

Targets CUAs at the **operating-system level** (not just web agents), and scores attacks with a **fully deterministic oracle** — checks on filesystem, service, and permission state — rather than an LLM judge.

## Key Results (with numbers)

- **Claude Opus 4.8:** ASR up from 4% (hand-written baseline) to **24%** with composed principles + feedback.
- **Gemini 3.5 Flash:** ASR up from 0% to **28%**.
- The **benign task still completes** — the attack doesn't break the agent's primary job (stealth).
- **Cross-model transfer:** principles discovered against one model transfer to a different architecture with no additional feedback.

## What's Novel

First self-improving, black-box IPI red-teamer for OS-level computer-use agents, with a deterministic (non-LLM-judge) scoring oracle and demonstrated cross-architecture transfer of learned attack strategies. The 0→28% jump on Gemini shows hand-written benchmarks massively understate adaptive-adversary risk.

## My Connection (to Manny's work)

This is a ready-made template for Manny's red-team automation: compose attack principles, run against the target, mine failures for new strategies, re-apply. The deterministic oracle design (filesystem/service/permission checks instead of LLM judges) is exactly how we should score agent red-team runs to keep results reproducible. Transferable strategies mean one red-team session can inform evals across many models.

## What I Learned (plain English)

A red team that learns from its failures is dramatically more effective than one with a fixed attack list — and the learned lessons carry across different models. Also: if you score attacks with an LLM judge, you're trusting the thing you're attacking; state-based oracles (did the file appear? did the service start?) are the honest way to measure.
