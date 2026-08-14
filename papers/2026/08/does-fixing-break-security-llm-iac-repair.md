# Does Fixing Break Security? An Empirical Study of Security Degradation in Iterative LLM-Driven Infrastructure-as-Code Repair

- **Paper:** [arXiv:2608.13404](https://arxiv.org/abs/2608.13404)
- **Authors:** Benjamin Agyekum, Fabio Santos
- **Published:** 2026-08-13
- **Code:** Not listed in abstract

## Problem Statement (plain English)

The standard way to use LLMs for Infrastructure-as-Code (IaC) is a feedback loop: generate Terraform/CloudFormation, run a validator (Checkov, terraform validate), feed errors back, and let the model repair them — repeatedly. Everyone measures whether the *best state ever seen* got better, which by definition never decreases. Nobody looked at what happens *between* iterations: does the model fix one problem while quietly breaking a security check that previously passed? This paper asks exactly that — how often does iterative LLM repair degrade security while fixing other issues?

## Methodology (technical)

- **Dataset:** 5,968 scenario timelines from the IaC-Eval benchmark; each scenario runs up to 5 repair iterations.
- **Configurations:** 15 total (6 model-specific RAG, 9 model-aggregated non-RAG, at 3 temperatures each) → 4,440 iteration transitions with Checkov data on both sides.
- **Tracking:** 30 individual CIS Benchmark check IDs; root causes classified from code diffs.
- **Detection modes:** standard (inclusive — a check failing on *any* resource counts) vs strict (exclusive — check failures only).
- **Metrics:** per-transition regression = a previously-passing CIS check that fails after a repair iteration.

## Key Results (with numbers)

- **Standard detection:** 13.8% of scenarios (24.8% of transitions) exhibit at least one security regression.
- **Strict detection:** 3.3% of scenarios (5.2% of transitions) — most apparent regressions are multi-resource measurement artifacts.
- **Root cause:** resource restructuring dominates at 79.0%.
- **Regression signature:** regression transitions show 2.6× more code churn (Cohen's d = 0.90) and 4.9× higher strict-mode check volatility (d = 1.49).
- **Self-correction:** 36.6% of standard-mode regressions self-correct within an average of 1.2 iterations.
- **Stopping point:** iteration 3 is the optimal stopping point.

## What's Novel

First study to examine the *raw per-iteration security trajectory* of LLM-driven IaC repair instead of cumulative-best metrics, which are non-decreasing by construction and therefore hide regressions entirely. Provides the first defensible, conservative regression rate (≈3.3%) plus an actionable iteration budget.

## My Connection (Manny's work)

Every AI red-team engagement that uses an LLM "fix loop" (generate → validate → repair) inherits this blind spot: final-state assessment misses mid-loop regressions. This is a direct red-team technique — instrument per-iteration CIS diffs, not just cumulative results, and treat iteration 4+ as a security-regression zone. Also a reminder that *"the model fixed it"* claims need per-check verification, and that resource restructuring (79% of regressions) is where a malicious or careless model hides a security downgrade.

## What I Learned (plain English)

"Best state" reporting makes AI repair loops look monotonic when they aren't. Roughly 1 in 4 repair steps breaks a previously-passing security check (inclusive view), and even the conservative estimate says 1 in 20. The fix is boring and mechanical: track every check across every iteration, stop at iteration 3, and treat big refactors as a security event.
