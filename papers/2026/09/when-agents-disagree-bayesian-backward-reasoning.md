# When Agents Disagree: Bayesian Backward Reasoning as a Label-Free Anchor for Multi-Agent Collective Decision-Making

**Authors:** Ken Chen, Wei Wang, Sachith Seneviratne, Hansani Weeratunge, Saman Halgamuge
**Institution:** University of Melbourne (Dept. of Electrical & Electronic Engineering / CS)
**Published:** Sep 10, 2026
**Link:** https://arxiv.org/abs/2609.11709 | **Code:** not linked in abstract

---

## Problem Statement (plain English)

When you run several LLM agents on the same question and they give conflicting answers, *how you combine them* decides whether their diversity helps or whether they just pile up the same mistake. Today's aggregators — majority voting, electoral rules, LLM-as-judge — all share a hidden flaw: they reason in one direction only (evidence → label), so the agents' errors are correlated and the mixture inherits them. The paper asks: can a *differently structured* estimate break that correlation and give a better anchor for deciding who to trust?

## Methodology (technical)

- **Forward vs. reverse posteriors:** Standard aggregation combines forward estimates — each agent maps evidence to a label. The authors additionally construct a **reverse posterior** per instance via **Bayesian backward reasoning** from an explicit likelihood. Forward and reverse posteriors are *differently factorized* approximations of the same underlying posterior.
- **Cross-path consistency:** Because differently-factorized estimates tend to fail in different places, the divergence between an agent's forward and reverse posteriors is informative. They use **Jensen-Shannon (JS) divergence** to rank agents by cross-path consistency.
- **Three aggregation strategies built on that signal:**
  - **MinJS** — hard selection: pick the agent with minimum JS divergence.
  - **FwdJS** — soft reweighting of agents by the signal.
  - **LogLin** — log-linear fusion of the forward and reverse evidence.
- **Optional calibration:** When labeled data is available, a lightweight **two-stage calibration** refines the reverse anchor further.
- **Evaluation:** **DDXPlus** medical-diagnosis benchmark across **five LLM backbones**.

## Key Results (with numbers)

- **MinJS outperforms random selection across all five backbones.**
- **FwdJS generally beats the strongest baseline.**
- **LogLin achieves the best overall performance** among evaluated methods, with its **largest gains on the subset where agents disagree** — precisely the hard cases.
- The reverse posterior has **weaker standalone accuracy**, yet serves as a *more useful anchor* than forward-only alternatives — i.e., it is valuable for its complementarity, not its solo correctness.
- Two-stage calibration with labels further improves aggregation.

## What's Novel

1. **Label-free trust signal for multi-agent aggregation:** you can rank/weight agents without ground truth, using the agreement between two independently-factorized inference paths.
2. **Reframing disagreement as structure, not noise:** divergence between forward and reverse reasoning is treated as a consistency probe rather than a defect.
3. **Reverse posterior as an anchor** that helps most where forward-only methods fail — the disagreement subset.

## My Connection (to Manny's work)

Multi-agent systems are exactly where a single compromised, poisoned, or simply wrong agent is hardest to spot. This paper gives a cheap, **label-free consistency check** you can run as a *defense*, not just an accuracy trick: if one agent's forward and backward reasoning diverge sharply from the panel, that agent is a candidate for quarantine — useful for detecting a prompt-injected or hijacked subagent in a swarm without needing a ground-truth oracle. It also pairs naturally with this week's agent-swarm misbehavior incidents: consensus mechanisms that assume "more agents = more truth" are demonstrably exploitable, and cross-path consistency is one concrete, testable alternative.

## What I Learned (plain English)

Asking the same model to reason *backward* from a conclusion gives you a second opinion that tends to fail in different ways than its forward reasoning. Comparing the two lets you spot which agent in a crowd is least trustworthy — without ever knowing the right answer — and that is a much better way to combine a panel than plain voting.
