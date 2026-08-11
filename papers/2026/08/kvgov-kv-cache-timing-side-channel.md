# Governing the KV Cache: Preventing Timing Side-Channel Leakage in Multi-Tenant LLM Inference

- **Paper:** [arXiv:2608.09225](https://arxiv.org/abs/2608.09225)
- **Authors:** Tejasvi C. Addagada
- **Institution:** Not stated on the arXiv page (single author, independent submission)
- **Published:** 2026-08-10 (v1) — 12 pages, 5 figures, 9 tables
- **Subjects:** cs.CR, cs.AI
- **Code:** Experimental scripts and raw results available from the author on request (no public repo).

## Problem Statement (plain English)

LLM inference servers share a key-value (KV) cache across tenants to speed up repeated prefixes. Sharing creates a timing side channel: by probing cache-hit latency, an adversarial tenant can reconstruct another tenant's private prompt. Three published attacks (PROMPTPEEK, EarlyBird, InputSnatch) hit up to 100% success against unprotected vLLM and SGLang. This paper builds a governance layer that closes all three attack families at once.

## Methodology (technical)

- **KVGov:** a governance layer over the prefix-cache path. Core mechanism: a per-principal salt σ_p = HMAC_K(secret, principal_id) seeds the block-hash chain, making cache keys *cryptographically disjoint across principals* — tenant A's cached blocks can never be hit by tenant B's probes, killing the timing oracle.
- **ORIGAMI:** a Stackelberg water-filling audit scheduler that allocates audit effort against adversarial probing, reducing adversary expected utility.
- **Evolutionary stability analysis:** identifies an adversary-prevalence tipping point below which global caching remains stable (i.e., caching benefits survive even with some attackers present).
- **Ablation:** N=1000 trials, deterministic judges, isolating the per-principal salt as the necessary-and-sufficient component.

## Key Results (with numbers)

- Addresses all three published attack families (PROMPTPEEK, EarlyBird, InputSnatch) under one mechanism.
- Ablation (N=1000) isolates the per-principal salt as necessary and sufficient.
- ORIGAMI reduces adversary expected utility by 12.6% at realistic tenant heterogeneity (Gini 0.63).
- Stability analysis: 31.6% adversary-prevalence tipping point below which global caching remains stable.
- Hardware validation (Qwen2.5-7B-Instruct, vLLM 0.26.0, NVIDIA A100): gate-verified cold/cached TTFT ratio of 0.22, confirming the channel exists and is measurable pre-fix.

## What's Novel

Most KV-cache side-channel work is attack papers; this is a *defense* that unifies all three known attack families under one primitive (per-principal key salting) rather than patching each attack. The Stackelberg audit scheduler and the evolutionary stability analysis (when does caching remain viable with attackers present?) are unusual, game-theoretic angles for inference-infrastructure security.

## My Connection (to Manny's work)

- Any multi-tenant LLM deployment (shared MCP gateways, managed model APIs, internal inference platforms) inherits this channel — prompt confidentiality is not just a data-at-rest problem.
- The per-principal salt pattern is a clean design rule for agent platforms that share caches or prompt prefixes across users/tenants.
- Useful audit question: does the inference stack bind cache keys to tenants, or does prefix reuse leak cross-tenant timing?

## What I Learned (plain English)

Performance optimizations that share state between users (like KV-cache prefix reuse) quietly create side channels that leak what other users are typing. The fix isn't to disable the optimization — it's to make the shared state *cryptographically partitioned* per tenant, and to think about attackers' incentives (how many attackers can be present before the optimization stops paying off).
