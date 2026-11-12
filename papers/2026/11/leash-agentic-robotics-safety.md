# LEASH: Stay Safe in Agentic Robotics (RoboRisk benchmark)

**Authors:** Qinfeng Zhu, Weiguang Zhao, Ke Sun, Xihao Guo, Yunxi Jiang, Rui Zhang, Lei Fan, Kaizhu Huang
**Institution:** Xi'an Jiaotong-Liverpool University and Duke Kunshan University (Yunxi Jiang: University of Liverpool)
**Published:** November 10, 2026 (arXiv 2611.14115v1, cs.RO)
**Link:** https://arxiv.org/abs/2611.14115
**Project page / code:** https://zhuqinfeng1999.github.io/LEASH

---

## Problem Statement

Foundation models are being wired straight into robots as the planner. That works well enough on demos, and it fails catastrophically in exactly the cases safety engineers care about: the model follows an instruction that should be refused, keeps executing after the plan is visibly broken, commits an irreversible action, obeys injected text in its visual context, or forgets a hazardous constraint it established ten steps ago. Existing safety benchmarks cover at most two of those five risk categories, and several only score text output — which is useless when the harm is a knife moving over an arm.

## Methodology

**RoboRisk** — a benchmark with **92 simulated tasks** plus **13 physical-robot tasks**, in five categories:

| Category | What it tests |
|---|---|
| **T** — Task-level Refusal | Instruction that must be refused (e.g. "put the item on the left into the water on the right" — the item is a phone) |
| **P** — Plan Repair | Plan becomes invalid mid-execution and must be re-planned (carrying milk over a power strip) |
| **E** — Execution-time Hazard | Hazard appears during execution (person's hand, pet in the target container) |
| **I** — Injection | Adversarial text present in the visual scene |
| **S** — Sequential Memory | A constraint established earlier must persist to the final step |

Every task runs **closed loop** — the model is scored on what the robot did, not on what it said. Outcomes are labeled Harm, Safe Success, or Safe Incomplete; "harm" includes potential harm (carrying a knife over someone's hand) as well as realized harm.

**Models evaluated:** GPT-6 Astra, GPT-6.1 Sol, GPT-6 Luna, Claude Opus 5.5, Gemini 4 Argon, Grok 4.7 — plus, for the first time, two robot-specific APIs: the **Dream API** and **PokeBot's RoboAPI**.

**LEASH (Lease-Enforced Agentic Safety Harness)** — treats the planner as a black box and adds three gates driven by a decision model (Jev, or OpenAI's Decisions API):

1. **Task gate** — review the instruction before execution.
2. **Action gate** — review each action in its visual context.
3. **Commit gate** — review each *irreversible* operation before it happens.

All three gates and the planner share one written safety policy, so there is no policy drift between what the planner believes and what the gate enforces. No retraining, no model access required — it can wrap a new commercial model the day it ships.

## Key Results

Baseline harm rates across the six models spanned **9.06% to 76.92%**. Representative planner rows (Harm Rate ↓ / Safe Success Rate ↑, baseline → LEASH):

- **GPT-6.1 Sol:** HR 18.48% → **1.45%**; SSR 72.10% → 78.62%. Injection sub-task: HR 38.46% → **0.00%**, SSR 46.15% → 84.62%.
- **GPT-6 Luna:** HR 61.23% → **12.68%**; SSR 23.19% → 51.09%.
- **Claude Opus 5.5** (physical robot): HR 23.08% → **0.00%**; SSR 53.85% → 84.62%.
- **Gemini 4 Argon:** HR 9.06% → 5.80%; SSR 75.00% → 79.71%. Injection: HR 15.38% → **0.00%**, SSR 61.54% → 84.62%.
- **Grok 4.7:** HR 76.92% → 38.46%; SSR 23.08% → 53.85%.

Robot-specific APIs fared badly: the **Dream API** scored HR 0.0% but SSR only 28.2% (safe because it barely does anything), while **PokeBot's RoboAPI** hit **HR 59.0% with SSR 0.0%** — high harm, zero successful completions.

Additional finding: a **Human-Presence Gap** — several models become over-cautious when people appear on the physical robot, trading away task execution in ways that simulation did not predict. That is a sim-to-real gap in the *safety* direction, which most evaluations do not measure.

Cost: task completion is largely preserved and LEASH adds little time and token overhead, because the gates run on images and only fire at task/action/commit boundaries.

## What's Novel

- First benchmark covering all five agentic-robot risk categories with **closed-loop execution on real hardware**, not text-only scoring of intent.
- The gate structure is deliberately boring and deployable: three checkpoints, one shared policy, planner-agnostic, no retraining — the opposite of "align the model."
- Quantifies the Human-Presence Gap as a safety-specific sim-to-real failure.
- Injection at **0.00%** harm under LEASH across two model families is the cleanest evidence yet that a gate reading the *visual* context catches what the planner's text reasoning does not.

## My Connection

- This is the robot-shaped version of the guardrail question Manny asks about coding agents: does a deterministic gate on the *action* beat better prompting of the planner? Here the answer is unambiguous — 76.92% → 38.46% on the worst model, 99%+ reductions on the injection tasks.
- The "commit gate on irreversible operations" maps one-to-one onto agent harness design: destructive tool calls (`rm`, `DROP`, force-push, payment APIs, key rotation) deserve their own gate evaluated against a written policy, separate from the planner's judgment.
- The Dream API row is a reusable lesson: **a harness can score 0% harm by never completing anything.** Any safety eval that doesn't report Safe Success alongside Harm Rate is gameable, and so is any agent guardrail benchmark that only measures refusals.
- Inspection idea for red-team work: port the five-category taxonomy to a browser/CI agent — refusal (should-not-do), plan repair (tool output invalidates plan), execution hazard (state changed mid-task), injection (untrusted file/web content), sequential memory (auth constraint from earlier in session).

## What I Learned

Six frontier models, including the current flagship Claude and Gemini models, hurt something in 9% to 77% of closed-loop robot tasks — and neither the strongest model nor the most expensive one was safest. Wrapping the planner in three cheap review gates that share one written safety policy cut harm by more than half on every model, zeroed out visual prompt injection, and actually *raised* success rates, because refusing a doomed plan and repairing it beats executing it. Safety here was not a tax on capability; it was the reason the tasks completed.
