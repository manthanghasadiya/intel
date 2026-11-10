# ShadowBench-MCP: Measuring Shadow Invocations in MCP-Style Tool Routing

**Authors:** Eyhab Al-Masri, Wenjun Yang
**Institution:** Not listed in the abstract
**Published:** Submitted November 9, 2026 (arXiv:2611.11720) — **Presented at the 35th USENIX Security Symposium (USENIX Security 2026)**
**Link:** https://arxiv.org/abs/2611.11720
**Code:** Not stated in the abstract

---

## Problem Statement

MCP-style agents are routers. Given a task, an LLM picks among tools, APIs and software-supply-chain workflows. That makes the **execution path itself a security-relevant model output** — more relevant, arguably, than the generated text that most safety work inspects.

The threat the paper names is **shadow invocation**: a *task-preserving request variant* that causes the router to select a **riskier execution path** — one that skips validation, provenance checks, staging, redaction or audit controls — **even though the request never asks for anything malicious.**

This is not prompt injection. Nothing adversarial is planted in the environment. The user's intent is unchanged. Only the *phrasing* shifts, and the router takes a shortcut it was implicitly allowed to take because it was optimized for efficiency.

Prior MCP security work has focused on malicious servers, tool poisoning, and injection. This asks a different question: **what does routing policy alone cost you?**

## Methodology

**The metric suite.** ShadowBench-MCP separates three things that are usually conflated, then combines them:

- **TPSS** — task preservation: how well does the variant keep the original goal intact
- **OPSS** — operational pressure: what efficiency/cost signal is the variant exerting
- **RGAS** — route-geometric drift: how far the selected path moved in routing space
- **SODS** — the combined **pre-execution** score (the important property: you can compute it *before* any tool fires)

**The study.** A focused policy-ablation on a **Groq-hosted qwen/qwen3-32b router**: 120 routing decisions yielding **96 original-versus-variant comparisons** across **six security-sensitive workflows**. The treatment is routing policy — efficiency-first vs. policy-aware — with everything else held fixed.

## Key Results

- **Efficiency-first routing: 29.2% shadow-invocation rate**, with a **26.7% safety-tool skip rate**. Roughly one in three requests took a riskier path, and one in four skipped the safety control — with no attacker involved.
- **Policy-aware routing: 4.2% and 1.7%** respectively.
- That's an **85.7% relative reduction in shadow invocations** from changing routing policy alone.
- **Pre-execution signals distinguish shadow routes at ROC AUC up to 0.94** — i.e. you can flag these before the tool call fires, not after.

The authors are explicit about scope: this is **early, single-router evidence of a policy-dependent routing-layer security effect, not a claim of universal vulnerability.** One model, six workflows, 96 comparisons.

## What's Novel

1. **Naming and isolating the phenomenon.** "Task-preserving request variant that changes the path" is a genuinely new category — distinct from injection (no adversary content), from tool poisoning (no malicious server), and from jailbreaking (the goal never changes).
2. **Pre-execution scoring.** SODS is computable before the tool runs. Most agent security metrics are post-hoc; a pre-execution gate is deployable.
3. **Treating routing policy as a security configuration.** The finding that a policy *setting* moves the shadow rate from 29.2% to 4.2% reframes "which model routes best" as "which routing policy do you run."

## My Connection

This is the third independent measurement in this digest of the same underlying claim, and the cleanest one: **the risk is in the path, not the prompt.** Harness Matters says unsafe behavior happens with no attack content; StealthyPoison says a skill can be weaponized while behaving normally; ShadowBench says a router takes the unsafe path 29.2% of the time purely because it was tuned for efficiency.

For Manny's harness work, the practical implication is uncomfortable and specific: **any cost or latency optimization on the routing layer is a security change** and needs to be evaluated as one. If you've recently tightened a router to "pick the cheapest sufficient tool," you may have shipped a 7x increase in safety-tool skips without touching a single security control.

The 0.94 AUC number is the deployable part — a pre-execution classifier over the request+route pair, gating on SODS rather than on content inspection.

## What I Learned

Shadow invocation is what "the agent was just being efficient" looks like when you measure it. The user said "quickly," the router interpreted that as "skip the staging step," and no policy violation was technically committed because nobody wrote down that staging was mandatory. The fix is not a better classifier — it's making the safe path the *cheap* path, so efficiency optimization and safety point the same direction.
