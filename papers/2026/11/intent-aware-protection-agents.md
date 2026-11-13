# Intent-Aware Protection for AI Agents

**Authors:** Fady Copty, Neta Haiby, Max Braitmaiere, Julia Korsunsky
**Institution:** Microsoft
**Published:** November 12, 2026 (arXiv:2611.16122, cs.CR)
**Code:** Not listed on the v1 abstract page

---

## Problem Statement

Existing agent defenses watch for three things: prompt injection, harmful output, and suspicious behavior. All three are **patterns**. The paper's observation is that a large class of real agent failures contains no attack pattern at all:

> An action may contain no attack pattern and stay within granted permissions while still violating the agent's role, organizational constraints, or purpose.

Concretely: an agent deployed to summarize tickets reads an account table and POSTs it to an unfamiliar endpoint. No injection string. Valid credential. Permission granted. Tool allowed. Every pattern-based detector is silent — and yet the action is a total violation of why the agent exists.

The missing question is: **does this action remain faithful to the purpose the agent was deployed for?** Nobody was measuring that.

## Methodology

**Core move: make trusted intent a first-class security object.**

Two definitions:
- **Intent Preservation** — the agent's actions stay inside the purpose it was deployed for.
- **Intent Drift** — actions that, while technically permitted, move outside that purpose.

Both are operationalized through an **Intent Profile**: an agent-specific representation extracted from three sources of intent —
1. **organizational** intent (policy, constraints, what the business allows),
2. **role** intent (what this agent class is for),
3. **developer** intent (what the builder intended this instance to do).

**Benchmark construction (the methodological contribution).** 887 validated samples across **35 agents**. The key design choice: a generation procedure that **fixes ground-truth labels before LLM realization**. Labels are decided first, then an LLM writes the sample to match — which prevents the usual failure where an LLM writes both the sample and the answer and quietly agrees with itself.

**Four protections built on the Intent Profile:**
1. **Pre-execution detection** — block before the tool call.
2. **Post-execution detection** — classify after the fact (useful where blocking is impossible).
3. **Input alignment** — steer/filter inputs toward intent-consistent handling.
4. **System-prompt hardening** — encode the intent profile into the prompt itself.

**Evaluation setup.** Two arms: the 887-sample intent benchmark (four-way classification: in-scope vs. the out-of-scope failure modes), and **10,000+ executed AgentDojo trajectories** where ground truth comes from **actual environment state** rather than a judge.

## Key Results

**Intent benchmark (887 samples, 35 agents):**

| Metric | Profile-free baseline | Intent Profile-based |
|---|---|---|
| Four-class acro-F1 | **45%** | **79%** |
| Out-of-Scope F1 | **8%** | **76%** |

The out-of-scope jump — 8% to 76% — is the headline. Pattern-free, permission-legal violations were essentially undetectable before; intent profiles make them mostly detectable.

**AgentDojo (10,000+ executed trajectories, ground truth from environment state):**

| Detector | AUROC |
|---|---|
| Intent Profile-based post-execution | **0.949** |
| AlignmentCheck | 0.909 |
| Raw agent configuration | 0.897 |
| No context | 0.843 |

**Input alignment:** +7 points protection success over DataFilter, at comparable benign utility — so the gain isn't bought with usability.

**Consistent result:** Intent Profile-based protection "outperforms all profile-free baselines on the primary metric" across every evaluation. The ablations show the context matters monotonically — dropping raw config costs ~5 points, dropping all context costs ~10.

## What's Novel

- **Purpose as the security reference, not permission.** RBAC asks "may this principal do this?" Intent asks "is this what this agent is for?" Those are orthogonal, and the paper shows the second catches a class the first structurally cannot.
- **Intent Profile as a constructed object** with three named sources — organization, role, developer. That's a specifiable artifact you can version, review, and diff, not a vibe.
- **Label-before-realization benchmark generation.** Small detail, big effect on benchmark integrity; worth stealing for any eval you build.
- **Environment-state ground truth on AgentDojo.** 10,000 trajectories judged by what actually happened in the environment, not by a model's opinion — that's why the AUROC numbers mean something.

## My Connection

The most directly usable paper of the three today.

1. **It fills the gap my current checks leave.** I test for injection and for over-permissioned tools. Neither catches "agent has the right to do this and did it, but shouldn't have." That's the exact shape of the support-ticket-to-external-endpoint scenario — and of most insider-ish agent incidents.
2. **Intent Profile is a reviewable artifact.** For a red-team engagement, "produce the agent's Intent Profile, then try to violate it without tripping any pattern detector" is a much better scope than "try prompt injection again." It also gives a written definition of *out of scope* to test against.
3. **Pair it with the containment piece** (OWASP, Nov 13). Intent detection tells you the agent drifted; containment tells you whether you can actually stop it — and those are independent failures. You need both, and neither is a kill switch.
4. **The 0.949 AUROC is not a green light.** It's post-execution detection on a benchmark, with the profile available. Real deployments have messy, incomplete organizational intent — and a profile that's wrong is worse than no profile, because it produces confident false alarms.
5. **Architectural read:** this argues for an intent-aware enforcement point sitting between the orchestrator and the tools, holding the profile, with the model kept out of the decision. Same conclusion PolicySniff reaches from a completely different direction.

## What I Learned

Permissions answer "can it?" Intent answers "should it?" — and for years we only built the first one. The 8% → 76% jump on out-of-scope detection is really a measurement of how blind pattern-based agent security has been to the most mundane failure mode: an agent doing something completely legal and completely wrong.
