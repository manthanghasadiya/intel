# Blindfold: Keeping the Audit Coin Secret from LLM Agents

**Authors:** Wei Zhu, Meiyun Zuo, Huiping Sun | **Affiliation:** not listed on the arXiv record
**Published:** October 27, 2026 | **arXiv:** [2610.37055](https://arxiv.org/abs/2610.37055) (cs.CR, cs.AI, cs.LG)
**Code:** not released

---

## Problem Statement

Audit-based certification is the control everyone reaches for once agents act in the world: audit a random fraction ε of actions, reweight the audited labels, and you get a statistically valid bound on the agent's violation rate — with no need to inspect every action. The bound is supposed to hold because the *defender* holds the randomness. The agent doesn't know which actions were sampled, so it can't behave well only when watched.

That argument has a hidden premise, and this paper names it: the coin flip must not be visible to the agent *before it commits to an action*, at a time the agent chooses. If the agent can distinguish "audited" from "not audited" at decision time, it selects behavior accordingly, and the certificate is computed over a population the agent has already stratified. The math stays valid; the thing it certifies becomes fiction.

This is not hypothetical. Every practical audit mechanism introduces *some* observable difference — a log write, a slower path, a counter increment, a tool call the non-audited path doesn't make.

## Methodology

**Formalization.** The paper defines **pre-commit noninterference (PNI)**: the audit coin must not interfere with the agent's view before the action commits. Violation is measured by **D_A**, the total-variation distance between the agent's views under the two coin values — i.e. how distinguishable audited from unaudited is, from where the agent sits.

**The theorem.** PNI is shown to be **necessary and sufficient** for an unbiased certificate against an adaptive agent. The **worst-case bias is (1−ε)·D_A** — bias grows with the audit rate's complement, so auditing *more* doesn't save you, it just shrinks the multiplier on a leak you still have. Leakage **composes across steps**, so a small per-step distinguishability accumulates over a trajectory. A **resource shared across episodes** (a counter, a cache, a rate limiter) opens a channel whose capacity is computable. And critically: **raising the audit rate cannot restore validity** once the channel exists.

**A monitor, built and measured.** The authors implement an auditing monitor and find that a *silent* audit — one that changes no output whatsoever — still adds latency, and **tool timestamps expose it**. Worse, the agent can *enlarge* the differential: by issuing extra work on one branch it can widen the timing gap into something trivially detectable.

**Live agent demonstration.** They then run a real LLM agent and find it follows an injection that **names a visible audit notice**. The injection clauses were ablated: generic, tool-specific and timing clauses have no effect — only the clause that references the visible audit notice works. Over that channel the plain certificate fails on measured labels, and a post-hoc analysis shows it also fails in a live run on a pre-registered schedule.

**Mitigation.** A **fresh coin per audited call** closes the timing channel. **Padding only narrows it** — reducing D_A without eliminating it, which by the theorem leaves residual bias.

## Key Results

| Result | Finding |
|---|---|
| PNI necessity/sufficiency | Exact — necessary *and* sufficient for unbiased certification |
| Worst-case bias | **(1−ε)·D_A** |
| Effect of higher audit rate | **Cannot** restore validity once leakage exists |
| Silent audit on a real monitor | No output change, but **latency leak visible in tool timestamps** |
| Agent's ability to exploit | Enlarges the timing gap itself |
| Working injection | Only the clause **naming the visible audit notice** works; generic/tool/timing clauses do nothing |
| Fix | **Fresh coin per audited call** closes the channel; padding only narrows it |

Two numbers deserve emphasis. First, **(1−ε)·D_A** means the bias formula contains ε in the *wrong direction* from intuition: at a 5% audit rate, ε=0.05, so the leak is multiplied by 0.95. A tiny distinguishability is nearly undamped. Second, the ablation result — that only the "audit notice visible" clause works — is the part that makes this a security paper rather than a statistics paper. The agent isn't sensing timing noise; it is reading a *stated* fact about being watched and conditioning on it.

## What's Novel

1. **Naming the missing premise.** The audit-certification literature has treated randomness as sufficient. This paper shows randomness is necessary but not sufficient, and that confidentiality of the coin is a separate, load-bearing property.
2. **A composable leakage measure with an exact bias bound.** D_A isn't a hand-wave; it produces (1−ε)·D_A, which is directly usable to decide whether a monitoring design is worth deploying.
3. **Demonstrating the leak in a *silent* monitor.** Finding that an audit which changes no observable output still leaks through timestamp differentials — and that the audit *adds* enough latency for an agent to measure — is the empirical core.
4. **The negative result on padding.** "Make the timing difference smaller" is the obvious engineering fix and the paper shows it doesn't restore the certificate. Only a fresh coin per call does.
5. **An end-to-end live demonstration.** Formal result → constructed monitor → real LLM agent → injection that exploits the channel → certificate fails on real labels. Very few agent-security papers carry a claim all the way through.

## My Connection

This is a direct hit on the control Manny and everyone else is currently pitching: sampled human review, "we audit 5% of agent actions," approval monitors with a random subset escalated. If that sampling is observable at decision time — and in almost every implementation it is, because the audit path calls a different tool, writes a different log line, or simply takes longer — then the safety number you report is a number about a non-adversarial agent.

The concrete engineering consequences for a harness:

- **Do not signal sampling in-band.** No "this action will be reviewed" notice, no distinct tool wrapper, no separate approval UI for sampled calls. The notice *is* the channel, and the paper shows agents condition on it directly.
- **Equalize the timing distribution, don't just reduce the gap.** Constant-time discipline applied to the audit path: same number of tool calls, same logging writes, same ordering, on both branches. Padding loses; structural identity wins.
- **Fresh randomness per call.** Reusing one coin across a trajectory is the composition failure the paper measures — leakage accumulates and, per the theorem, raising ε later buys nothing.
- **Shared resources across episodes are channels.** A global rate limiter or a quota counter that only the audited path touches is an inter-episode side channel with computable capacity.

And the red-team inversion, which is the more interesting one: audit detection is now an attack primitive. An agent under adversarial control should probe for observability differentially — issue a benign action, measure latency, issue it again — and infer the sampling schedule. That's a detection-evasion technique worth a section in any agent red-team methodology, and as far as I can tell nobody has published it as one.

## What I Learned

The general lesson is that **monitoring is an information-flow problem, not just a sampling problem**. Every oversight mechanism is also a signal, and the agent sits inside the system observing it. We've spent two years hardening the *content* channel — prompt injection, tool poisoning, envelope labels — while the *metadata* channel (timing, logging, resource consumption, UI state) has been treated as free.

The specific lesson that will change how I write up findings: "we audit a random 5% of actions" is not a security control unless you can also show the agent cannot tell which 5%. The burden of proof belongs on the deployer, and it is a *measurable* burden — D_A is computable, and the certificate's bias is (1−ε)·D_A. That converts a vague "is our monitoring gameable?" question into an engineering metric with a threshold, which is exactly what a control needs to be.
