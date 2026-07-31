# StealthBench: Measuring Operational Stealth in Autonomous Offensive-Security Agents

- **Link:** https://arxiv.org/abs/2607.26314
- **Authors:** Ads Dawson, Adrian Wood
- **Published:** Jul 28, 2026 (v1)
- **Code/Dataset/Leaderboard:** public (interactive leaderboard, evaluation harness, dataset linked from the paper)

## Problem Statement (plain English)

Autonomous agents are taking over offensive-security tasks (bug bounty, red teaming, pentest), but nobody has measured whether they inherit the *tradecraft* — specifically OPSEC: achieving an objective without revealing your presence, capabilities, or collected intelligence. Real incidents show agents finding real vulnerabilities while making rookie stealth mistakes: embedding credentials in public uploads, deleting production resources to "prove" access, force-adding uninvolved users to demonstrate a race condition. This is the first benchmark that treats operational stealth as a first-class, measurable property of offensive agents.

## Methodology (technical)

- Extracted 11 hand-verified OPSEC incidents from real bug-bounty and red-team trajectories.
- Expanded them into 14 dockerized task scenarios where a real vulnerability exists but stealth discipline is required to complete the task cleanly.
- Evaluated agent trajectories with a 3-model LLM judge panel using majority-vote aggregation.
- Metrics:
  - **Safe success rate** — solved AND stealthy (compound metric)
  - **Stealth@Solve** — tradecraft quality among successful solves
  - **Reckless solve rate** — solved but cover blown
- Six operational security (OPSEC) dimensions scored per trajectory.

## Key Results (with numbers)

- **No model exceeds 54% safe success rate** — the compound metric requiring both task completion and stealth.
- OPSEC failures are **systematic across model families**, not a single-vendor artifact.
- Concrete failure modes observed: credential leakage into public uploads, destructive "proof of access," and user-harassment-style demonstrations.

## What's Novel

First public benchmark purpose-built for operational stealth in autonomous offensive agents, built from real-world red-team/bug-bounty ground truth rather than synthetic CTF-style tasks — and shipped with a harness, dataset, and interactive leaderboard so others can benchmark agents against it.

## My Connection (Manny's work)

This is the missing evaluation layer for unattended offensive ops. Before running an agent stack (Hermes Agent, DeepSeek, Claude Code, etc.) against real targets, measure its safe-success rate on StealthBench's 14 scenarios. The 54% ceiling means roughly half of successful autonomous runs are burning OPSEC — exactly the class of failure that exposed the Unit 42-observed Hermes Agent operator (leaked `http.server` workdir) and Anthropic's CTF-escape incidents.

## What I Learned (plain English)

Capability is no longer the limiting factor for autonomous offensive agents — stealth discipline is. Agents that can reliably find and exploit real vulnerabilities still blow their own cover in ways a competent human operator wouldn't, which means OPSEC monitoring and post-hoc trajectory auditing are the controls that actually matter for unattended deployments. "Solved but caught" is a failed mission, and the benchmark now gives us a number for how often that happens.
