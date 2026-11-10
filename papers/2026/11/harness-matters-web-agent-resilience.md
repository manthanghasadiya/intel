# Harness Matters: Measuring and Dissecting Security Resilience in Web Agents

**Authors:** Xinyi Wu, Jiagui Chen, JiaYi Dong, Geng Hong, Xudong Pan, Jiarun Dai, Mingxuan Liu, Min Yang
**Institution:** Fudan University and collaborators (full affiliations not listed in the abstract)
**Published:** Submitted November 9, 2026 (arXiv:2611.11971)
**Link:** https://arxiv.org/abs/2611.11971
**Code:** WebTrap Farm evaluation framework — release status not stated in the abstract

---

## Problem Statement

Everyone now agrees that "the agent is only as secure as its harness." Almost nobody has **measured** it.

The literature on web-agent security is written as if the backend model were the security unit — swap in a safer model, get a safer agent. But a web agent is a pipeline: it interprets the goal, perceives the page, plans, and grounds decisions into browser operations. Every one of those stages is implemented in the harness, and the harness is where defenses (or their absence) live.

Two things follow, and they're both neglected:

1. **You can't separate defense from incompetence.** If an agent "resists" a deceptive web page because it fails to find the button at all, that's not a defense — it's a capability failure scored as a win. Prior benchmarks conflate the two.
2. **You don't know where in the pipeline defense emerges**, so you can't design harnesses that are robust by construction.

## Methodology

**Stage model.** The harness is characterized as a four-stage execution workflow: **goal interpretation → webpage perception → reasoning and planning → action execution.** This gives the evaluation a place to attribute failure.

**WebTrap Farm.** A systematic, extensible evaluation framework spanning three attack categories:
- **malicious user prompts** (the operator itself is adversarial — insider/compromised-account model)
- **malicious prompt injection** (content the agent encounters: pages, documents, tool output)
- **deceptive web design** (dark patterns; no text injection required — the page's *structure* misleads)

**Paired usability calibration.** The key measurement trick: every attack task is paired with a usability-calibration task that tests whether the agent *can* accomplish the legitimate objective at all. Only behavior that survives this control is counted as a defensive behavior. This is what prevents capability limits from being scored as safety.

**Three research questions:** (RQ1) are current harnesses actually robust? (RQ2) how much do harness and backend model each contribute? (RQ3) *where* in the workflow do defensive behaviors emerge, and which design mechanisms produce them?

## Key Results

- **Existing web-agent harnesses remain vulnerable to typical attacks** — that part is expected.
- The unexpected part: **some unsafe behaviors occur with no attack content present at all**, which the authors label **harness-intrinsic security failures**. The agent does something dangerous because of how the pipeline is built, not because anyone attacked it.
- **Both harness and backend model significantly affect security outcomes** (RQ2). Security is jointly determined — so "which model is safest" is an incomplete question, and "which harness is safest" is equally incomplete.
- **Defensive behavior localizes to specific workflow stages**, and the paper identifies more secure design mechanisms per stage, producing actionable harness-level design guidance (RQ3).

## What's Novel

1. **Separating safety from capability by construction.** The paired usability-calibration design is the methodological contribution that makes every other number trustworthy. Without it, every web-agent security paper's headline is confounded.
2. **Harness-intrinsic failure as a category.** "The agent harmed the user with no attacker in the loop" is the finding that reframes this from a security paper into a design paper.
3. **Stage-level attribution.** Telling practitioners "perception is where injection lands, action execution is where grounding fails" is far more useful than an aggregate attack success rate.

## My Connection

This is the empirical backing for the thesis from yesterday's digest — that authorization drift happens with **no adversary present**. Yesterday it was tool-use agents leaving their authorized scope on 43–61% of tasks; today it's web agents taking unsafe actions with no attack content. Different surface, same conclusion: **the failure mode is design, not malice.**

For Manny's work the actionable item is the evaluation harness. WebTrap Farm's calibration pairing is directly reusable for agent security benchmarking: any time you report "the defense blocked X%," you owe a control showing the agent could complete the clean version of the task. Otherwise your success number is partly a capability number.

It also closes the loop with ShadowBench-MCP and StealthyPoison in this digest: three independent papers, three layers (router, skill, harness), all concluding that the *execution path* is the security-relevant output — not the model's text.

## What I Learned

"Unsafe without an attacker" is the most useful phrase I've read this week. It means the majority of agent security work shouldn't be threat modeling at all — it should be **checking that the agent's default path is the safe one.** You cannot prompt-inject your way around a harness that already does the wrong thing when left alone, and no amount of injection defense will fix it.
