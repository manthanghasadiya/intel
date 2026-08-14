# LLM-Assisted Dynamic Threat Analysis for Attacker-Reachable Software Weaknesses in Autonomous Vehicles

- **Paper:** [arXiv:2608.13450](https://arxiv.org/abs/2608.13450)
- **Authors:** Md Wasiul Haque, Sagar Dasgupta, Mizanur Rahman, Md Rayhanur Rahman
- **Published:** 2026-08-13
- **Code:** Not listed in abstract

## Problem Statement (plain English)

Autonomous vehicles run huge safety-critical software stacks where a weakness reachable from adversarial input could affect steering or braking. Static analysis finds candidate weak spots, but confirming that one is *actually exploitable* requires building executable test harnesses — a slow, expert-only job. Can LLMs automate the whole pipeline: turn static-analysis findings into compilable, fuzzable harnesses that dynamically confirm exploitability?

## Methodology (technical)

- **Target:** Autoware, an open-source autonomous-driving stack.
- **Static analysis:** compiler-precise across 185 packages → 1,375 decision rules, 2,274 validation checks, 482 input-to-safety-output flows → weakness taxonomy → 740 reachable sites sampled.
- **Generation:** two local open-weight LLMs (a reasoning model and a code-specialized model), a no-static-context ablation, and a naive-template baseline produce 3,700 harness artifact sets.
- **Loop:** artifacts compiled against the real build under sanitizers, repaired via compiler-in-the-loop feedback, fuzzed when executable.

## Key Results (with numbers)

- **80% of first-shot compilation failures come from dependency wiring, not program logic.**
- The reasoning model compiled 64% of harnesses on the first attempt vs 6% for the code-specialized model.
- Repair reached full object-compileability for the reasoning model only through extensive stubbing; fewer than half of its harnesses reached the fuzzer.
- All 37 observed crashes originated in **stubbed code**, not Autoware.
- **No candidate weakness was dynamically confirmed within budget.**

## What's Novel

A build-integration failure taxonomy for LLM-assisted dynamic analysis, and a rigorous negative result: for full AV stacks, the barrier is build integration — not candidate generation and not fuzzing. Also a clean comparison showing reasoning models vastly outperform code-specialized models at harness construction.

## My Connection (Manny's work)

Directly relevant to LLM-driven exploitability triage: when automating "is this finding exploitable?", the expensive failure mode is environment/build wiring, not the LLM's analysis. Budget accordingly (stubbing, containerized builds) and distrust crash reports that originate in stubbed code — a classic false-positive trap in LLM security pipelines. Also a good template for red-team tooling: static analysis → LLM harness generation → compiler-in-the-loop → fuzzing.

## What I Learned (plain English)

LLMs are great at *suggesting* an exploit harness and bad at making it build inside a real project — and when you stub things to make them build, your fuzzing results become meaningless. "The LLM found a crash" needs a provenance check: did the crash come from the target or from the scaffolding the LLM invented?
