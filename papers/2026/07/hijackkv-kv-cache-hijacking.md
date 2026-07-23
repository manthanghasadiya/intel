# Paper: HijackKV — New Threat in Position-Independent KV Cache Reuse

- **Authors:** Yichi Zhang, Zhiqi Wang, Huan Zhang, Yuchen Yang
- **Institution:** (not listed on arXiv page)
- **Published:** July 22, 2026
- **Link:** https://arxiv.org/abs/2607.19957
- **Venue:** USENIX Security 2026 (accepted)
- **Code:** Not publicly linked

---

## Problem Statement (Plain English)

Modern LLMs use KV caching to speed up inference — storing intermediate computations so they don't have to recompute everything for every new token. Newer "position-independent" cache designs let you reuse cached KV entries even when the same text appears at different positions in a sequence (e.g., caching the KV for "the capital of France" once and reusing it wherever it appears). The problem: if an attacker can poison a cached KV entry with their own malicious context, and that poisoned entry gets reused in another user's query, the model will behave as if the attacker's words were part of the victim's input — even though the attacker's text never appears in the input. This paper formalizes and demonstrates this attack.

## Methodology (Technical)

The authors introduce **HIJACKKV**, the first systematic attack framework against position-independent KV cache reuse:

1. **Threat Model:** Attacker controls a prefix that gets KV-cached. The KV entries for a benign-looking token chunk are computed with the attacker's prefix prepended. When the victim's query hits the cache (because it contains the same benign chunk), the poisoned KV is returned, carrying the attacker's context.

2. **Optimization:** The attacker optimizes their prefix so that the KV computed for a subsequent common benign text (e.g., "the", "and", common phrases) encodes the attacker's desired behavior, while the text itself remains unchanged and recognizable for future cache hits.

3. **Attack pipeline:** Optimize attacker prefix → Inject into shared cache → Wait for victim query to trigger cache reuse → Model behaves according to attacker's hidden context.

4. **Evaluation:** Tested across multiple LLM families and architectures. Measures success rate (whether hijacked model behavior matches attacker intent), robustness under probabilistic cache-hit scenarios, multi-turn persistence, and black-box transferability.

## Key Results (with numbers)

- **94% average success rate** in a single attempt across tested configurations
- Effective under **10% cache hit rate** (sparse reuse scenario)
- Survives **50% frequent recomputation** (half of cached entries are recomputed from scratch)
- Persists over **multi-turn interactions** (poison carries across conversation turns)
- **Transfers across models** in black-box settings (poison optimized on one model works on another)
- Accepted at USENIX Security 2026 — top-tier security venue

## What's Novel

- First attack framework targeting position-independent KV cache reuse specifically
- Prior KV cache attacks focused on stealing or evicting data; this is the first to show **cache poisoning for behavior hijacking**
- The poisoned KV vector approach means **no attacker text appears in the input** — makes it invisible to input-filtering defenses
- Demonstrates practical exploitation under realistic constraints (partial cache hits, mixed recomputation)

## My Connection (to Manny's Work)

KV caching is ubiquitous in LLM serving infrastructure. If Manny's red-team assessments cover any service using prompt caching, semantic caching, or KV reuse (common in MCP server backends, AI agent platforms, and model-as-a-service providers), this attack vector needs testing. The attack's invisibility to input filtering makes it particularly relevant for adversarial evaluation of MCP servers and agent orchestration systems.

## What I Learned (Plain English)

KV caches are supposed to just speed things up by storing intermediate math results. But because those math results encode *context* (not just token identity), reusing them is like pasting text from a document you didn't read — the formatting, meaning, and intent come with it. This paper shows you can craft a "sticky note" that attaches to common words like "the" or "and" and, when the cache reuses those words, the sticky note's content silently changes the model's behavior. The defense is to either not reuse KV across different users (defeating the performance benefit) or to sanitize/isolate KV cache entries by origin — neither is trivial in production.
