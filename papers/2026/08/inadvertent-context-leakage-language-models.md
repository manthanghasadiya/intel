# Inadvertent Context Leakage in Language Models

- **Paper:** https://arxiv.org/abs/2608.19857
- **Authors:** Jaiden Fairoze, Neal Mangaokar, Kamalika Chaudhuri, Sanjam Garg, Saeed Mahloujifar
- **Published:** 2026-08-20
- **Code:** Not listed in abstract

---

## Problem Statement (plain English)

AI agents hold sensitive context — calendars, credentials, health records, financial data — to be useful. The question: does the mere presence of a secret in the context window change the model's *benign* outputs in ways that let an attacker reconstruct the secret, even when the model correctly refuses direct extraction? And can an attacker actively engineer prompts to amplify that leakage, turning the model into a covert carrier?

## Methodology (technical)

- Controlled experiments across eight proprietary models.
- Two settings: (1) passive leakage — secrets in context, ordinary non-adversarial requests, outputs analyzed for reconstruction; (2) active leakage — adversarial prompt engineering to amplify the effect.
- Novel adaptive attack assuming black-box API access only.
- Two practical attack demos: a trained classifier inferring semantic predicates about user memories (health conditions, financial events) from routine outputs, and an RL-trained adversary extracting full Social Security Numbers from a production-style agent.

## Key Results (with numbers)

- 2-digit in-context secrets reconstructed with near-perfect accuracy; 4-digit secrets at 82% exact match — from outputs to ordinary, non-adversarial requests.
- Leakage scales with capability: stronger instruction-following amplifies sensitivity to in-context secrets. Leakage is a byproduct of capability, not a patchable bug.
- Full SSN extraction demonstrated from a production-style agent via RL-trained adversarial prompts.

## What's Novel

- Shows leakage is *inadvertent* — it happens in outputs the model produces anyway, without any extraction request.
- Establishes that refusal to reveal secrets is not a defense: the correlation is in the benign text stream.
- The capability-scaling result (better models leak more) reframes leakage as structural, not fixable by alignment tweaks.

## My Connection (to Manny's work)

For any red-team engagement against agentic systems: this is a reliable, low-noise exfiltration channel that bypasses "the model refused" as a control. Red teams should test client agents for context-leakage channels (send secrets into context, ask benign questions, train a classifier on outputs). Defenders can't rely on refusal or output filtering alone — the mitigation is architectural: minimize what goes into context (least-privilege context), which aligns with AWS's "agent as orchestrator, not gatekeeper" pattern from today's digest.

## What I Learned (plain English)

- "The model won't tell you the secret" is not a security boundary — secrets bleed into ordinary outputs.
- Capability and leakage grow together; you can't patch your way out.
- Context minimization is the real control: if the secret never enters the context window, it can't leak.
