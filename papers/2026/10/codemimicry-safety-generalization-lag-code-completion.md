# CodeMimicry: Exploiting Safety Generalization Lag via Structured Code Completion

**Authors:** Zhen Liang, Hai Huang, Wentao Chen
**Institution:** Not stated in abstract
**Published:** September 30, 2026
**Link:** https://arxiv.org/abs/2609.39902
**Code:** Not stated in abstract

---

## Problem Statement (plain English)

Safety alignment for LLMs is trained mostly on **natural language**, but the models are also excellent at **code**. The paper identifies a "safety generalization lag": the refusal behavior learned in prose does not transfer to the code domain, creating a *code-completion blind spot*. If a malicious request is dressed up as ordinary, syntactically valid code, the model will happily complete it.

## Methodology (technical)

- **Threat frame:** *safety generalization lag* — alignment tuned on natural language fails to transfer to structured/code domains.
- **Attack:** CodeMimicry, a fully automated **black-box** jailbreak. It generates **structured, object-oriented code prompts** that induce harmful output *through the code-completion path* rather than a chat refusal path.
- **Baselines:** compared against template-based and optimization-based jailbreak baselines.
- **Mechanistic analysis:** latent-space representations, projection onto refusal-related directions, and activation steering — to explain *why* code embeds bypass refusal.
- **Scale:** 8 state-of-the-art **commercial** LLMs.

## Key Results

- **96.25% attack success rate** across 8 commercial LLMs.
- **1.51 queries on average** — extremely efficient, black-box only.
- Significantly outperforms both template-based and optimization-based baselines.
- Mechanistic analysis shows code prompts move away from refusal directions in latent space, and activation steering corroborates the mechanism.

## What's Novel

- Names and formalizes **safety generalization lag** as a distinct failure mode (alignment that doesn't cross the NL→code boundary).
- Demonstrates a **black-box, high-ASR, low-query** exploit of that gap — no white-box access needed, so it's directly deployable.
- Couples the empirical result with a **mechanistic** explanation (refusal-direction projection / activation steering), not just a score.

## My Connection (to Manny's work)

- Directly relevant to **agentic coding tools**: any coding agent (Claude Code, OpenClaw, Cursor) that completes or executes model-authored code inherits this blind spot.
- The "security-critical logic written in code" angle connects to real bugs (missing auth checks, unsafe deserialization) generated via the same completion path — a plausible exploit-primitive source.
- The mechanistic framing (refusal directions, activation steering) overlaps with the counter-injection / activation-steering defense line seen elsewhere in the agent-security literature, making it a useful attack/defense pair.

## What I Learned (plain English)

A model that refuses to *describe* something dangerous will still *write the code* for it, because its safety training never really covered the code channel. Wrap the bad request as a class or a function and the refusal doesn't fire. That's the whole exploit — cheap, black-box, and it works on nearly every commercial model tested. The fix isn't more prose-based alignment; it has to be alignment that generalizes into structured domains.
