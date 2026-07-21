# Adaptive Adversaries: A Multi-Turn, Multi-LLM Benchmark for LLM Agent Security

**Authors:** Devina Jain, David Hartmann, Chuan Li
**Institution:** Not explicitly stated (independent research / workshop paper)
**Published:** July 20, 2026
**Link:** https://arxiv.org/abs/2607.18063
**Code:** Released with benchmark (21 evaluation scenarios, 10 dev scenarios, orchestrator, baseline harnesses, multi-attacker CLI)

---

## Problem Statement

LLM-based agents process external content, making them vulnerable to prompt injection and multi-turn manipulation. Existing safety benchmarks evaluate defenders against a **fixed pool of attacks** collected before evaluation — either single-turn or multi-turn. But real attackers adapt: they observe defender responses and pivot their strategy across rounds. There is no benchmark that measures defender robustness against **adaptive, multi-round attacks** where the attacker learns from each interaction.

## Methodology

The authors construct a benchmark with **21 scenarios** where:

- **Defenders** are memoryless LLMs — each response is evaluated as a fresh interaction (no conversation state carries over between turns, preventing the defender from learning across rounds but also preventing attackers from poisoning history).
- **Attackers** are autonomous LLMs that observe prior defender responses and adapt their attack strategy across multiple rounds (up to 15 rounds).
- **Structured-output scoring** is used to evaluate whether the attack succeeded.
- **Three frontier LLM attackers** are pooled: Claude Opus 4.6, GPT-5.4, and Gemini (unnamed variant).

The experimental design isolates the effect of multi-turn adaptivity:
1. Restrict scoring to the first attacker turn only → measure baseline ASR
2. Allow 15 rounds of adaptive attack → measure multi-turn ASR
3. Pool all three attackers → measure ensemble coverage
4. Run 3×3 attacker-defender matrix → measure pairwise differences

A large open competition ("gpt-oss-20b") was also run, producing **18,422 battles** used for final scoring rounds.

## Key Results

| Condition | ASR Range |
|-----------|-----------|
| Single-turn (first turn only) | 0–1% |
| 15-round adaptive attack | 5.4–14.0% |
| Pooling three frontier attackers | 1.4–2.2× unique successful attacks vs best single attacker |

- **Claude Opus 4.6** and **GPT-5.4** tied in aggregate (5.4% each, overlapping 95% CIs)
- **But weaknesses differ sharply**: on one specific scenario, Opus reached **60% ASR** (95% CI 36–80%) while GPT-5.4 and Gemini stayed at **7%** (CI 1–30%)
- **13 of 21 scenarios** distinguish at least one defender pair
- Defender rankings disagree across scenarios (Kendall's W = 0.19 — very low concordance)
- Generated attacks have **low cosine similarity (0.02–0.14)** to attacks in existing benchmarks — they find genuinely novel vectors

**Released artifacts:**
- 945 transcripts from the 3×3 frontier matrix
- 18,422 gpt-oss-20b competition battles
- Attack-replay dataset
- 21 evaluation scenarios + 10 development scenarios

## What's Novel

1. **First benchmark for adaptive multi-round attacks** against memoryless LLM defenders — the attacker adapts per round based on defender responses
2. **Multi-attacker pooling**: demonstrates that no single LLM attacker is sufficient for robust testing — ensemble attacks are 1.4–2.2× more effective
3. **Low cross-benchmark similarity**: the attacks found are structurally different from existing benchmark attack vectors
4. **Defender ranking disagreement**: the "best" defender depends on the scenario — a single aggregate score is misleading
5. **Open competition framework**: 18K+ battles provide a rich dataset for future research

## My Connection

This is directly relevant to Manny's mcpsec work. MCP servers are the attack surface for LLM agents — the very class of systems this benchmark evaluates. Key takeaways:

- **Single-turn testing is insufficient**: Manny's current red-teaming methodology likely tests single-shot prompt injection. Adaptive multi-round attacks are ~10× more effective.
- **Ensemble attackers find more vulnerabilities**: Manny should test MCP servers against at least 2–3 different frontier LLMs as attackers, not just one.
- **Scenario-specific weaknesses**: The finding that defender rankings vary by scenario means Manny should build a diverse scenario set rather than optimizing for aggregate metrics.
- **The memoryless defender design** is notable: MCP servers are typically stateless per-request, matching this setup. The 5.4–14.0% ASR represents real risk for MCP-based agent systems.

## What I Learned

- Adaptive multi-turn attacks are **an order of magnitude more effective** than single-turn attacks (0–1% → 5.4–14% ASR)
- No single frontier model is sufficient for red teaming — you need an ensemble
- Defender rankings are **highly scenario-dependent** — a model that's "best" overall may be catastrophically bad on specific scenarios (Opus at 60% ASR on one scenario vs 7% for GPT)
- The attack vectors found are **qualitatively different** from existing benchmarks (cosine similarity 0.02–0.14), suggesting existing defenses are blind to these attacks
