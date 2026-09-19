# The More It Says, the More You Pay: A Black-Box Audit of Provider-Side Token Inflation in LLM Services

**Authors:** Leilei Chen, Lan Zhang, Chen Tang, Pengcheng Sun, Jiewei Lai, Yixiao Huang, Zhaopeng Zhang, Xinpeng Shen (8 authors)
**Institution:** not stated in abstract (industry/academic security group)
**Date:** Submitted 17 Sep 2026 (announced in the Fri, 18 Sep 2026 cs.CR batch)
**Link:** https://arxiv.org/abs/2609.20370
**Code:** not indicated (single-probe audit methodology described in detail)

---

## Problem Statement (plain English)

In pay-per-token LLM services, **the more the model says, the more you pay**. That gives a dishonest provider a direct incentive to secretly make models ramble — pad outputs to inflate the token bill — while keeping output good enough that the customer doesn't notice. The paper defines and audits this: a **Provider-Side Token Inflation Attack (PTIA)**. The hard part is that a customer only sees black-box responses; they have no trusted reference model and no clean historical baseline to compare against.

## Methodology (technical)

- **Attack taxonomy:** five representative PTIA instances spanning the provider-controlled pipeline at the **query, prompt, representation, and model levels** — i.e. injections at multiple stages of generation.
- **Black-box the audit:** the authors' key observation is **PTIA saturation** — trace it to stopping behavior at the end-of-sequence token.
  - An *initial* PTIA sharply lowers the EOS token probability (longer outputs).
  - *Further* strengthening or composing attacks lowers it only marginally, so output length saturates.
- **Single-probe audit** built on that insight: apply a controlled *lengthening* intervention as a probe. Under PTIA, the probe induces far fewer additional tokens than under normal service (because the model is already saturated); the gap is the signal.
- The audit needs **no trusted local reference model** and **no historical clean responses**, and its separately issued original and probed requests resemble ordinary traffic, making evasion difficult.

## Key Results (with numbers)

- Each of the five attacks increases **mean output length to more than 10.2x the clean baseline** — demonstrating PTIA's financial appeal and feasibility at multiple generation stages.
- Single-probe audit performance:
  - **85.1% average detection rate** across four open-weight models.
  - **False-positive rate below 2%.**
- Applied to **15 real LLM API services**, the audit **flags 7 for PTIA-consistent behavior**.
- Saturation is traced mechanistically to the **end-of-sequence token probability**, with an initial attack causing a sharp drop and subsequent intervention only a marginal one.

## What's Novel

1. Formalizes a **provider-as-adversary** economic attack on pay-per-token billing — a threat model customers rarely consider.
2. Turns the attack's own weakness (saturation) into the defense, yielding a **lightweight single-probe audit** that needs neither a reference model nor historical data.
3. Designs the probe requests to **mimic ordinary traffic**, making the audit hard for a provider to detect and selectively evade.
4. Provides **field evidence**: 7 of 15 real services flagged, moving PTIA from theory to a measurable real-world phenomenon.

## My Connection (to Manny's work)

- If you ever assess LLM providers or route agent traffic through third-party APIs, this is a **direct audit method** you can run with a handful of probe requests — no access to the provider's stack required.
- The "provider is in the trust boundary" framing pairs with agent-security work: the LLM API is an untrusted component, and token counts are part of what you must verify.
- The 10.2x inflation ceiling and sub-2% FPR numbers put concrete bounds on a cost/reliability risk that often gets waved off as "just pricing."

## What I Learned (plain English)

The people billing you per word have a motive to make the model wordy. You can't see their internals, but you *can* notice that padding a response stops working once they've already padded it — so send a request designed to force extra output and measure how much extra you get. Normal service gives you a lot more; a padded service gives you almost none. That simple probe caught 7 of 15 real APIs. It's a reminder that the model endpoint itself belongs in your threat model, not just the code around it.
