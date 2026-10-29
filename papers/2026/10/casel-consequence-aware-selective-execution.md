# CASEL: Consequence-Aware Selective Execution for Reliable Long-Horizon Tool-Using Agents

**Authors:** Yanyan Zhang, Fei Shen, Ting Yan, Hao Zhang
**Published:** October 27, 2026
**Link:** https://arxiv.org/abs/2610.36570
**Code:** Not stated in the listing

---

## Problem Statement

Current agent runtime safeguards are wrong about *which* tool calls deserve scrutiny. Two common heuristics dominate: route on generic model uncertainty scores, or route on coarse "does this call mutate state" heuristics. Both are proxies for the thing you actually care about.

The problem with uncertainty is that it's uncorrelated with consequence. A model can be deeply uncertain whether you meant `us-east-1` or `us-east-2` (harmless) and highly confident that it should delete the production table (catastrophic). The problem with mutation heuristics is that `rm -rf /` and appending a comment to a log file are both mutations.

The cost of getting this wrong runs both directions: over-gating sends everything to a human and destroys the value of having an agent, under-gating ships harmless-looking calls that aren't. This paper treats the allocation itself as the object of optimization.

## Methodology

**CASEL** (Consequence-Aware Selective Execution Layer) is a **model-agnostic runtime allocation policy**. For each proposed tool call, it picks exactly one of six execution modes:

1. Direct execution
2. Verification
3. Sandboxing
4. Clarification
5. Human approval
6. Abstention

**The decision inputs** — five features, and the choice of five is the argument:
- **Calibrated failure probability** — P(the call is wrong), calibrated rather than raw model confidence
- **State-dependent consequence** — what this specific call does *given current system state*, not a static severity label
- **Reversibility** — can the effect be undone
- **Ambiguity** — is the intent under-specified
- **Evidence freshness** — is the grounding the model used still current

**The selection rule.** Each mode gets a cost; CASEL picks the minimum. Cost functions penalize harm plus the cost of the intervention itself, so a heavyweight intervention on a cheap reversible call is itself penalized. This is what prevents the policy from collapsing into "escalate everything."

**Theory.** The authors formalize direct execution as *selective prediction*, then state and test the validity conditions for two mechanisms:
- A finite-sample threshold
- A trajectory-level risk ledger

**Evaluation.** A controlled simulation: 20 seeds, 120,000 test actions each. Critically, **every method gets the same intervention menu and the same compute cost** — so the comparison is about allocation policy, not about who bought more verification budget.

## Key Results

| Policy | Harmful-side-effect trajectories | Mean normalized harm | Human approvals |
|---|---|---|---|
| Uncertainty-ranked gate (baseline) | 14.75% | 0.184 | baseline |
| **CASEL** | **7.02%** | **0.123** | baseline |
| Failure-prob × consequence gate | 5.82% | 0.120 | **+45%** |
| CASEL without reversibility routing | 15.53% | — | — |
| Trajectory-level risk budget | 11.82% (worse) | — | equal cost |

**Three results worth sitting with:**

1. **CASEL halves harmful trajectories** (14.75% → 7.02%) at the same cost. That's the headline.

2. **The naive two-factor product beats CASEL on raw harm** (5.82% vs 7.02%) — but spends 45% more human approvals to do it. The authors report this rather than hiding it, and the honest read is that CASEL is buying harm reduction *per unit of human attention*, which is the actual constraint in deployment.

3. **Reversibility-awareness is load-bearing.** Removing it returns the harm rate to 15.53% — worse than the baseline it replaced. This is the paper's most surprising single number: one of the five inputs isn't a refinement, it's essential.

4. **Trajectory-level risk budgets make things worse** (11.82% at equal cost). Budgeting risk across a trajectory sounds prudent and empirically isn't, because it spends the budget early on low-consequence calls and then has nothing left for the call that matters.

## What's Novel

1. **Consequence as state-dependent, not static.** Most guardrail work treats "dangerous tool" as a property of the tool. CASEL treats it as a property of the tool *in this state at this moment*. `DELETE FROM users WHERE id=1` is routine; `DELETE FROM users` is not, and the tool is identical.

2. **The full six-mode action space.** Prior work mostly does binary gate/pass or a three-way allow/review/block. Making abstention and clarification first-class — and cost-comparable to sandboxing — is a better model of what a runtime actually can do.

3. **Cost-normalized comparison.** Equal intervention menu and equal compute for every method. This is rarer than it should be and it's why the numbers are trustworthy.

4. **Negative results reported as results.** Reversibility ablation and the risk-budget failure are both in the headline table.

**Stated limitation, which the authors are explicit about:** intervention effectiveness is *assumed, not measured*. The results rank allocation rules under the simulator's assumptions and do not establish deployment effectiveness. They say this directly in the abstract. Treat the percentages as relative rankings, not predictions.

## My Connection

- **This is the formalization of the decision Manny makes manually in every agent engagement.** When you're reviewing an agent's tool surface, the question "which of these calls should require a human?" is exactly CASEL's allocation problem, and the five inputs are a better checklist than intuition: is it reversible, how bad given current state, how ambiguous, is the evidence fresh, how likely is the model wrong.
- **The reversibility result is a directly testable finding.** If removing reversibility-awareness degrades a policy below baseline in simulation, then any client agent guardrail that doesn't model reversibility is probably worse than no guardrail at all. That's a concrete review item.
- **The "consequence is state-dependent" framing is the argument to make to clients** who've built a static tool allowlist. An allowlist can't express "this call is fine at 10am on staging and catastrophic at 3am on prod."
- **It pairs with the Log-To-Leak paper from today.** CASEL allocates scrutiny by consequence — and Log-To-Leak's attack is designed so the malicious call *looks* low-consequence. Which means CASEL's consequence model is itself an attack surface.

## What I Learned

The field has been trying to make agents safer by making them more uncertain-aware, and uncertainty turns out to be close to the wrong signal. What matters is: **if this call is wrong, what breaks, and can I put it back?**

The deeper lesson is about where safety logic belongs. Every approach that asks the model to be careful is asking the thing under adversarial influence to police itself. CASEL is interesting because the policy is model-agnostic and runs outside the model — it takes a proposed call and adjudicates it. That's the same architectural conclusion the MCP papers keep arriving at from a different direction: **the authorization decision has to sit in the host, not in the context window.**

The caveat I'd carry forward: these are simulator numbers under assumed intervention effectiveness. The ranking is the contribution, not the percentages. Anyone citing "7.02%" in a deck as an expected production outcome is misreading the paper.
