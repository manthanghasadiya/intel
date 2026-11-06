# Signed but Not Safe: Requirement Integrity in Multi-Agent LLM Handoffs

**Authors:** Xiaobo Chen, Genliang Zhu, Ziyuan Wang, Chu Wang
**Institution:** Not stated on the arXiv record
**Published:** November 5, 2026
**Link:** https://arxiv.org/abs/2611.07483
**Code:** not linked on the abstract page at time of writing

---

## Problem Statement

Enterprise agents increasingly decompose a user's request across subagents and merge the results. Every handoff is a lossy, untrusted serialization step: a subagent can summarize away a requirement, soften it, or ignore it entirely, and the orchestrator merging the results has no way to know whether the requirement it *thinks* it is enforcing is the one the user actually stated.

This is not prompt injection in the classic sense. Nothing malicious needs to be planted. The failure is structural — requirements decay across agent boundaries, and under adversarial pressure they decay in the attacker's favor. The paper's framing is precise: the question is whether the *requirement set* survives the handoff intact, and whether the merger's compliance decision can be checked by something other than another LLM.

## Methodology

A **preregistered** study (registered before data collection, with exactly scorable synthetic tasks — the design choice that makes the numbers credible).

**Three handoff formats compared:**
1. Free-text summaries
2. Structured JSON
3. **Signed requirement manifests**

**The signed-manifest pipeline:**
- The orchestrator extracts each requirement as a **verbatim quote plus a checkable predicate**
- Code signs the manifest with **HMAC-SHA256**
- **Deterministic code** — not a model — verifies the merger's decision against the predicates

**Evaluation:** Qwen3-8B, 1,440 trajectories, five threat models. Plus two ablations that isolate the interesting boundaries: an oracle-manifest condition (perfect extraction) and a pre-signing injection condition (attacker writes to the source text *before* extraction).

## Key Results

- **Unsafe commits under attack: 69.4% → 9.0%** with signed manifests (60.4 pp reduction, 95% CI [51.4, 69.4]), with **no net loss of conformant commits** relative to structured handoffs. Security gain with no utility tax is the headline.
- **Signing defeats in-transit tampering completely:** unsigned manifests committed violations in **79%** of mutated runs; signed manifests in **none**.
- **Deterministic verification beats LLM checking on identical trajectories:** caught **78.7%** of violating decisions vs **59.0%** for an LLM checker, with **7%** vs **50%** false alarms. The false-alarm gap (7x) is arguably more important than the detection gap — an LLM checker that cries wolf half the time gets disabled by operators within a week.
- **Oracle manifest removed all remaining unsafe commits.** Every residual failure traces to extraction, not verification.
- **Pre-signing injection is the live boundary:** a single adversarial line in the source text pushed signed-manifest violations from **14.6% to 37.5%**.

## What's Novel

1. **Treating requirements as signable data with predicates, not prose.** The verbatim-quote-plus-checkable-predicate representation is the actual engineering contribution — it converts "did the agent comply" from a judgment into an evaluation, which is what makes deterministic verification possible at all.
2. **Isolating signature value from representation value.** Because structured JSON is a separate arm, the paper can show that *signing specifically* (not just structure) is what kills tampering: 79% → 0% on mutated runs.
3. **The oracle ablation that locates the trust boundary.** By showing an oracle manifest reaches zero unsafe commits, the authors prove the residual risk is entirely upstream of the signature. That is a clean, falsifiable statement of where the problem actually lives.

## My Connection

This is the correct architecture for Manny's multi-agent work, with one sharp caveat.

The **deterministic verifier over signed predicates** is a pattern to adopt immediately: any place where an agent's output gates a consequential action, the gate should be code checking a signed, structured claim — not a second model reviewing the first. The 50% false-alarm rate of the LLM checker is the practical argument: LLM-on-LLM review does not survive contact with an on-call rotation.

The caveat is the pre-signing gap (14.6% → 37.5%). Signing is a **transport integrity** control, not an input integrity control. If a document the extractor reads is attacker-influenced, you have cryptographically certified a poisoned spec. Which means the extraction step needs the same treatment the paper gives the handoff: provenance on the source, and ideally the same authority-based check that the task-tampering benchmark argues for.

Together with this week's other papers, the chain is visible: **ingest is untrusted (07640), retrieval is untrusted (08325), handoff is untrusted (07483)** — and signing only helps at the third link.

## What I Learned

Signing a bad requirement gives you a very trustworthy bad requirement. The paper proves that cryptographic integrity fixes exactly one hop — the handoff between agents — and does it perfectly (79% to 0% tampering), while leaving the actual weak point untouched: whoever turned the user's words into the requirement list. The practical rule is to sign what crosses a boundary, verify it in code rather than with another model, and then spend the security budget on the extractor, because that is where the remaining 37.5% lives.
