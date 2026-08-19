# KeyPooling: Measuring Where LLM API Relay Paths Collapse Prompt Cache Isolation

- **Paper:** https://arxiv.org/abs/2608.17485
- **Authors:** Bowen Sun, Yixi Cai, Xiaogeng Liu, Zhengyue Zhao, Yinzhi Cao, Chaowei Xiao
- **Published:** Submitted Aug 18, 2026 (cs.CR)

## Problem Statement (plain English)

"API relay" services (OpenRouter and similar gateways) sell access to OpenAI/Anthropic models. They authenticate each customer separately, but often forward requests through shared upstream credentials. Providers (OpenAI/Anthropic) scope prompt caches to upstream principals and namespaces — so if relay customers share one cache identity, they can observe each other's cache state: whether a prompt was seen before, token-by-token timing, and reconstructed content. Prior work showed cache sharing existed; nobody knew *which* credential, pool, adapter, or nested hop actually controls the final identity — and therefore where the isolation failure lives.

## Methodology (technical)

- **KeyPooling measurement method:** traces customer identity through cache lookup and write, verifies runtime transformations, and tests one predicted identity component at a time (credential, pool, adapter, nested hop).
- Tested **five open-source gateways** connected to OpenAI and Anthropic under shared upstream credentials.
- Validated on production traffic via an **outcome-independent OpenRouter frame** (weekly, 80.5% of eligible token volume covered).
- Controlled recovery procedure on one production route: attempt to recover target positions without any target access.

## Key Results (with numbers)

- **None** of the five open-source gateways bound customers to upstream credentials by default; under a shared credential, **all five** exposed cross-customer cache reads against both OpenAI and Anthropic.
- Principal/namespace splits, pool associations, and adapter/nested-relay contrasts localized the controlling transformations.
- OpenRouter frame: cross-account reads found for **12 of 28 labels carrying 33.7% of volume**.
- Controlled procedure **recovered eight consecutive target positions without target access** on one production route.
- Cache granularity, routing, rate limits, attribution, and budget act as *conditions* for token-by-token recovery — not security controls.
- Proposed fix (identity domain enforced at final cache lookup/write) preserves most reuse at a **1.7–2.5% cost increase**.

## What's Novel

- First measurement that pinpoints *where* isolation collapses (which credential/pool/adapter/hop), not just that it collapses.
- Production-scale validation (OpenRouter) showing the issue is real beyond toy gateways, with a practical, low-cost defense contract.

## My Connection (to Manny's work)

Any organization using API relays/gateways for agent fleets inherits this: cross-customer cache reads are a side channel for prompt-content disclosure and even token-level reconstruction of other tenants' data. For Manny's AI-security practice, this is a concrete vendor-assessment checklist item — verify whether your gateway binds customers to upstream credentials, and demand provider-enforced identity domains.

## What I Learned (plain English)

- Authentication and cache identity are separate things: a gateway can authenticate customers correctly and still pool them into one upstream cache identity.
- "Rate limits and routing" look like protections but are just conditions an attacker tunes around.
- The fix is conceptually simple (force a per-customer identity into the final cache key) and cheap (1.7–2.5% cost) — which means there's no good excuse for vendors not to ship it.
