# Trajectory-Level Security Debt in LLM Coding Agents

**Authors:** Prateek Kumar Rajput, Abdoul Kader Kabore, Yewei Song, Melissa Tessa, Tailia Malloy, Jacques Klein, Tegawendé F. Bissyandé
**Institution:** University of Luxembourg (SnT)
**Date:** 2026-09-28 (arXiv v1)
**Link:** https://arxiv.org/abs/2609.35199
**Code:** Not linked in the abstract page at time of writing.

---

## Problem Statement (plain English)

A coding agent does not jump straight to a finished program — it passes through **hundreds of intermediate code states** before it submits a solution. If you only scan the final submitted artifact, you completely miss the security story: a dangerous pattern the agent wrote early on and later masked with a cosmetic change is invisible. The authors argue we need to measure security **along the whole trajectory**, the same way you'd track technical debt as it accrues — not just at the finish line.

## Methodology (technical)

- Introduces the **Security Debt Line Integral (SDLI)**: a metric that **accumulates SAST risk whenever the agent reaches a new best test-pass ratio** — i.e. it charges the agent for security risk taken on during every capability gain, not just the final state.
- Instantiated with **four static application security testing (SAST) tools** and applied to:
  - **830 passing SWE-bench runs**
  - **712 ProgramBench final workspaces**
  - **13 public MirrorCode trajectories**
- The two large populations use the **final-state special case** of SDLI (risk at the end only); MirrorCode provides full trajectories.
- Uses **CWE-class agreement across tools** as a consistency signal rather than treating any scanner finding as a confirmed vulnerability.

## Key Results (with numbers)

- **Two-tool CWE class agreement occurs in only 3.9% of SWE-bench runs** and **26.2% of the 80 ProgramBench runs passing ≥90% of official tests**.
- Excluding **three advisory-heavy CWE classes** drops the ProgramBench agreement rate to **6.2%** — a large chunk of the "findings" are scanner noise.
- The authors are explicit these are **scanner findings, not validated vulnerability rates**.
- **Same-task runs differ in their measured scores** — trajectory security debt is not deterministic.
- One reconstructed ProgramBench run **exposes persistent findings seeded by its first implementation write** — early choices echo through the whole trajectory.
- A repair case study reduces the scanner signal while preserving tested behavior, **but reveals sensitivity to equivalent API rewrites** (semantically identical refactors change the measurement).

## What's Novel

1. **Security as an integral over the trajectory**, not a point sample of the final artifact.
2. **Capability-aware charging** — risk is attributed to the moment of each improvement, mirroring how performance is measured.
3. **Honest scanner-agreement analysis** — quantifies how much of the signal is tool disagreement and advisory-heavy noise, which most "AI writes vulnerable code" claims skip.
4. **Reproducibility caveat by construction** — shows the metric is sensitive to equivalent rewrites, flagging a real limitation for anyone tempted to use it as a gating score.

## My Connection (to Manny's work)

This is the measurement backbone for any "agentic code review" or "secure-SDLC agent" content: it argues the review gate must watch intermediates, not just the diff the agent finally submits. It also provides an honest counter-narrative to sensational "LLM code is 40% vulnerable" headlines — tool agreement is low and much of it is noise. Useful both as a tooling idea and as a calibration lesson.

## What I Learned (plain English)

Where a coding agent's security problems live is in the *path* it took, not the destination — an unsafe pattern introduced on the first write can survive to the end. But the paper is refreshingly skeptical of its own numbers: different scanners disagree wildly, and even rewriting code the same way can change the score. So the metric is best used to *study* agent behavior, not as a hard pass/fail gate yet.
