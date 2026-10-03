# Actions with Receipts: Jointly Binding Claims, Evidence, and Execution for Replayable Tool-Agent Auditing

**Authors:** Miaobo Hu, Shuhao Hu, Xiaobo Guo, Xin Wang, Bokun Wang, Yina Sa, Daren Zha, Jun Xiao
**Institution:** Chinese Academy of Sciences (Institute of Information Engineering, CAS)
**Date:** Submitted 29 Sep 2026 · Announced 2 Oct 2026
**Link:** https://arxiv.org/abs/2610.00327

---

## Problem Statement

Tool-using agents happily surface **citations** and **execution logs** — yet both can be individually well-formed while the *critical association between them is never audited*: **is the claim shown to the user the same claim that the committed execution emitted, and is it supported by the cited source?**

Because that association is unverified, a **valid citation and a valid trace can be transplanted** across claims, actions, runs, or source versions — a "frankenstein" audit where every part is genuine but the assembly is a lie. This is the agent equivalent of a forged provenance chain, and it breaks the trust assumptions behind agent audit logs and RAG-grounded answers.

## Methodology

The authors introduce a **claim-anchored execution contract** that *jointly binds* four things:

1. the emitted claim,
2. its exact **source span**,
3. the **ordered execution prefix** that produced it,
4. the **source version and access state** observed by that execution.

Each **receipt** contains:
- an **emission anchor** that deterministically locates the claim inside a committed answer / claim-bearing action,
- source identifiers, offsets, hashes and quotes,
- a **domain-separated execution commitment**.

A **deterministic integrity verifier** reconstructs the bindings *before* semantic or task labels are joined. The design deliberately **separates an integrity plane from a pluggable support plane**, so structural validity is never used as a proxy for entailment (a well-formed receipt ≠ a true claim).

The contract exposes **seven independently testable properties**: claim-emission binding, source binding, ordered-execution binding, oracle separation, persisted-object replay, execution-rerun consistency, and version/access binding.

## Key Results

- **1,280 cross-object attacks → 1,275 detected (0.9961)** by the joint contract.
- **Ablation:** removing any single targeted property collapses its detection rate to **0.0156–0.0625** — each binding is load-bearing, not redundant.
- On an **independently adjudicated 384-pair split**, the conflict-aware **support guard** reaches **F1 0.8865** with **false acceptance 0.0729**.
- On **unseen failure families**: F1 **0.8679**, false acceptance **0.0938** — i.e. it generalizes to novel tampering modes.

## What's Novel

1. **Claim-anchored execution contract** — binds claim ↔ source span ↔ execution prefix ↔ source version in one verifiable object.
2. **Emission anchor**: deterministically pins a claim's location inside a committed answer, killing relocation/transplant attacks.
3. **Integrity/support plane separation** — formatting correctness can't masquerade as factual correctness.
4. **Seven-property contract + ablation** proving each property carries independent detection weight.
5. **Replayability**: persisted-object replay and execution-rerun consistency make audits reproducible, not just logged.

## My Connection (Manny's Work)

This is the **auditability/provenance layer** for tool agents — the complement to runtime defenses like ToolFence, PACE and AGATE. Where those *prevent* bad tool calls, "Actions with Receipts" makes the *after-the-fact* record non-repudiable and tamper-evident, which matters for incident response: if an agent is compromised, can you still trust its logs? The cross-object transplant threat is also a clean, testable red-team primitive against any agent that cites sources.

## What I Learned (Plain English)

Real citations and real logs don't prove anything together — an attacker can take a true quote and a true action log and glue them together so the story is false even though every part checks out. The fix is a receipt that hard-links *this claim* to *this exact source* to *this exact execution* to *this version of the file*, then verifies the linkage before anyone reads the content. Structure proves nothing about truth — but it does prove nothing was swapped.
