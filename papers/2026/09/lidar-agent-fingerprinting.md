# LIDAR: Who Is Behind the Harness? Fingerprinting LLMs through Agentic Behavior

**Authors:** Chuyi Wang, Xiaohui Xie, Tongze Wang, Fangchen Luo, Yong Cui
**Institution:** Not stated in the preprint metadata (Tsinghua-linked author group)
**Published:** 2026-09-23
**Link:** https://arxiv.org/abs/2609.28559
**Code:** Not linked in the abstract

---

## Problem Statement (plain English)

When you build a coding agent, the model is one component behind a *harness* — the scaffold of system instructions, controller logic, tool definitions and execution feedback that decides how the model behaves. Providers and enterprises quietly swap the model behind a harness: a cheaper model replaces an expensive one, a fine-tune replaces a base model, a local model replaces an API model. That substitution changes security-relevant decisions — does the agent verify its own edits? does it recover safely from a failed tool call? does it notice when the spec and the tests disagree?

Existing LLM fingerprinting tries to identify the model from its *text* or *token distributions*. Inside an agent, those signals are laundered through the harness: the system prompt and controller reshape the output, so text-based fingerprints don't transfer. There is no reliable way to answer "which model is actually running here?" from the outside.

## Methodology (technical)

LIDAR (**L**LM **ID**entification from **D**ecisions and **A**ctions at **R**untime) is an *active, black-box* fingerprinting method for coding-agent execution.

- **Active probing.** Instead of passively observing, LIDAR issues three *coding probe pairs*. Each pair presents a controlled situation that elicits a different, model-characteristic behaviour:
  1. **Post-edit verification** — does the agent re-check a change it just made?
  2. **Transient-failure recovery** — how does it react to a recoverable tool failure?
  3. **Specification–test conflict resolution** — which does it trust when the spec and tests disagree?
- **Feature representation.** Resulting trajectories are encoded at two complementary levels: *instance-level* features (what happened in this run) and *distribution-level* features (how behaviour varies across runs).
- **Identification.** A lightweight probabilistic identifier compares those features against clean reference traces. No access to weights, logits, or provider internals is required.
- **Evaluation.** 36 models from 7 families across 2 different agent harnesses, benchmarked against four existing fingerprinting / API-auditing baselines, plus ablations on the feature levels, probe pairs, and their controlled variants.

## Key Results

- High **Top-1 accuracy and MRR** for model identification across 36 models / 7 families / 2 harnesses.
- **Outperforms four existing** fingerprinting and API-auditing baselines.
- Ablations show all three probe pairs and both feature levels contribute — the signal is not carried by one dimension.
- The central empirical claim: **agent execution behaviour provides model-identity evidence beyond final outputs**, and that evidence survives the harness.

## What's Novel

1. Reframes LLM fingerprinting as a *behavioural, action-level* problem rather than a text/token-distribution problem — explicitly designed to survive harness mediation.
2. Uses **active probes** that target security-relevant dispositions (verification, recovery, conflict handling) rather than stylistic quirks.
3. Fully **black-box**: no weights, no logits, no provider cooperation — the realistic threat/audit model for a hosted agent.
4. Works **across harnesses**, which is what makes the signal attributable to the model rather than to the scaffold.

## My Connection (to Manny's work)

This is directly relevant to agent-supply-chain and model-provenance questions:

- **Substitution detection.** If a vendor, wrapper, or "AI gateway" claims to run one model but silently routes to another — a common pattern with routers, fallback tiers, and grey-market resale — LIDAR-style probing is the way to detect it from the outside.
- **Post-incident attribution.** After an agent incident, "which model made this decision?" is often unanswerable from logs. Action-level fingerprints plus traces give an attribution path when weights and logits are unavailable.
- **Red-team tradecraft.** The probe-pair design (force verification, force failure recovery, force spec/test conflict) is a reusable template for eliciting *disposition*, not just capability — useful for building evaluation harnesses that surface unsafe defaults.
- **Harness-vs-model confusion.** The paper gives a rigorous argument for something worth repeating in reports: behaviour you attribute to "the model" may in fact be the harness, and vice versa.

## What I Learned (plain English)

A model's identity isn't just what it *says* — it's what it *does* when you make it work. Inside a coding agent, the harness washes out the textual fingerprints we normally rely on, but the decisions leak anyway: whether it double-checks itself, how it handles a transient error, what it does when instructions and tests conflict. Watching those decisions over a few controlled probes is enough to tell 36 models apart from the outside, with no access to the model itself. The practical lesson is that *behaviour under controlled stress* is a durable, cheap identity signal — and if you can use it to identify a model, so can anyone trying to catch you swapping one out.
