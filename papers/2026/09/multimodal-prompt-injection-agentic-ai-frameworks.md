# MMPIBench: An Experimental Evaluation of Multimodal Prompt Injection Attacks on Agentic AI Frameworks

**Authors:** Viet K. Nguyen, Mohammad I. Husain (California State Polytechnic University, Pomona)
**Published:** 2026-09-08 (arXiv:2609.09404v1, cs.CR / cs.AI)
**Link:** https://arxiv.org/abs/2609.09404
**Code/Data:** benchmark and data released (per abstract)

---

## Problem Statement (plain English)

Agentic AI frameworks let a language model plan, keep memory, and call tools that touch real files, mail, and services. Most of these agents also *read images* — so an attacker can smuggle text into the agent's context without going through the user at all (put it in a screenshot, a photo's metadata, a QR code, a fake UI). The paper asks the question the field has been hand-waving: when an instruction arrives through a perceptual channel rather than typed text, does the agent actually act on it — and if so, how far does it get?

## Methodology (technical)

The authors build **MMPIBench**, a reproducible benchmark that:

- Delivers a fixed attack set through **six visual carriers**: OCR-embedded text, overlays, EXIF metadata, QR codes, fake interfaces, and hybrids.
- Instruments the agent to trace each injected instruction across stages: **perception → planning → tool call**, recording where it dies.
- Sweeps **720 runs** spanning **6 frameworks × 5 foundation models × 6 carriers × 4 attacker objectives**.
- Extends the same harness to **audio**, the only other raw perceptual channel current frontier models accept (4 attacker objectives retained).

Metrics distinguish **attempt rate** (model proposes to act) from **completion rate** (the tool call actually executes), which is the key methodological choice — reporting only completion hides most of the exposure.

## Key Results (with numbers)

- Visual attacks: **completion ≈ 1%** of runs, but **attempt ≈ 12.8%** — a >10× gap.
- The gap is closed **almost entirely at the planning step**: the model reads the injected instruction and declines to act.
- **The model dominates the framework.** One model **never attempted an attack** and recognized the injection in **59.7%** of runs; two others attempted in **23.6%**. Framework choice barely moved the needle.
- **Audio is much more exposed:** only 2/5 models ingest audio and only 3/6 frameworks deliver it, but where the signal lands, attacks complete in **49% of cells** — and **75%** for one model.

## What's Novel

- First benchmark to measure multimodal prompt injection **as a staged pipeline** (perception/planning/action) rather than a binary success.
- Separates **attempt** from **completion**, showing completion-only reporting systematically understates exposure.
- Quantifies the **model-over-framework** result and finds the planner as the natural defense chokepoint.
- Extends the analysis to **audio**, showing non-visual perceptual channels are narrower in coverage but far less defended.

## My Connection

This is the measurement layer for Manny's agent-harness red-teaming: it gives a reusable, reproducible way to score a harness on multimodal injection, and the "planning step is where attempts die" result tells you exactly where to place guardrails in the products and tools he builds/reviews.

## What I Learned (plain English)

Agents *see* injections constantly (12.8% of runs) but usually refuse to act (1% completion) — and that refusal is a property of the **model**, not the framework you wrapped it in. Vision is the well-covered channel; **audio is the soft spot**, completing attacks in nearly half the cells where it's even deliverable. Measure attempts, not just successes, or you'll badly underestimate your exposure.
