# What a CAGE 4 Score Measures

**Authors:** James Rice, Ji Hwan Park, Andres Kwasinski
**Institution:** Not listed on the abstract page
**Published:** November 10, 2026 (arXiv 2611.14068v1, cs.CR)
**Link:** https://arxiv.org/abs/2611.14068
**Code/artifacts:** Instrument, patches to both reference defenders, traces and checkpoints behind every number, plus a five-item disclosure checklist — released with the paper (per abstract)

---

## Problem Statement

CAGE Challenge 4 is *the* benchmark used to compare learned multi-agent cyber defenders. Papers report a CAGE 4 score, put it in a table, and the reader infers that a higher number means a better defender. The authors ask a narrower and much more uncomfortable question: what does that number actually measure? Their answer is that three things read off CAGE 4 — defender quality, the effect of an intervention, and recovery precision — all measure less than the tables assume.

Concretely: the simulator silently discards any action asserted for an agent whose previous action is still running, and still charges one point for a discarded `Restore` exactly as if it had run. The documented observation does not expose the "busy" flag, so any defender that wants to know whether its agent is idle has to reach into the environment's internals through its own wrapper. Both reference defenders, KEEP and H-MARL Expert, do exactly that.

## Methodology

- Reproduce KEEP and H-MARL Expert in a **pinned container** so results are byte-comparable to the released policies.
- **Ablation 1 — the busy check.** Disable the undocumented busy-flag read at evaluation time and measure the score delta, keeping the executed action sequence fixed.
- **Ablation 2 — supervisory harness.** Instead of letting defenders issue actions to busy agents, hold interventions until the agent is idle, and measure the point gain at the published setting.
- **Training-time ablation.** Retrain from scratch without the busy check and compare against the released reproduction interval across paired seeds, tracking when each recipe leads or trails.
- **Variance decomposition.** Reseed only the environment, not the policy, and decompose per-episode outcome variance into environment variance vs. the policy's own sampling draw.
- **Documentation audit.** Check five published comparison tables for whether they state the artifact, sampling scheme, criterion and adversary that would make their numbers comparable, and whether reported "recovery precision" is scored against a stated ground truth.

## Key Results

- Turning the busy check off at evaluation costs **KEEP 99.58 points per episode** and **H-MARL 145.98**, in both cases purely the charge for `Restore` actions sent to busy agents — with H-MARL's *executed* actions unchanged step for step.
- On a supervisory harness, holding interventions until the agent is idle is worth **25.3 points (KEEP)** and **27.6 points (H-MARL)** per episode at the published setting.
- In training, the check decides how a busy step enters the trajectory: trained without it, KEEP leads at 100 updates in all four paired seeds and **trails from 200 updates on in all four**.
- Continued to 2,000 updates on three seeds, the repaired recipe lands inside the released interval on one seed and short on two — **by 27 and 125 points**. That spread across training seeds is wider than the released reproduction interval itself.
- **88.9% of per-episode outcome variance is the policy's own sampling draw**, because the evaluation seeds the environment but not the policy. A reported mean is one draw.
- Across five published comparison tables, **none** states the artifact, sampling, criterion and adversary needed to make its numbers comparable.

## What's Novel

Benchmark audits usually attack task design or dataset leakage. This one attacks the *simulator's accounting*: a hidden mechanic that charges for actions that never execute, plus an observation gap that forces defenders to read private state. It then quantifies the consequence for published leaderboards and shows the reference implementations are only reproducible if you inherit their undocumented behavior. The variance decomposition is the sharper knife — it says most of what a single CAGE 4 run reports is not the defense at all.

## My Connection

- Any agent-cyber benchmark number Manny quotes or debunks — ExploitGym, Terminal-Bench, CAGE — should now be paired with the question this paper formalizes: *artifact, sampling, criterion, adversary*. The five-item disclosure checklist is directly reusable as a "is this benchmark claim real" filter.
- The failure mode generalizes to harness evaluation: if the harness holds or drops agent actions under load (queueing, rate limits, tool timeouts) and the scorer doesn't distinguish "executed" from "accepted", measured capability is inflated by exactly the amount of dropped work.
- Concrete test idea: run any agent harness under synthetic queue pressure and compare scored success vs. *executed* action count. If they diverge, the harness's score is a scheduling artifact.

## What I Learned

A benchmark score can be dominated by how the simulator silently handles a busy agent. Two reference defenders lost 100–146 points per episode when a check they inherited — but that the documentation never mentions — was switched off, and nearly 90% of run-to-run variation was the model's own randomness rather than the defense. If a paper reports a CAGE 4 number without stating its artifact, seeds and adversary, you cannot compare it to anything.
