# TrajMark: Ownership Attribution and Segment-Level Tamper Localization for Coding-Agent Trajectories

**Authors:** Bokang Zeng, Zheng Gao, Xiaoyu Li, Xiaoyan Feng, Jiaojiao Jiang (University of New South Wales, Sydney)
**Published:** 2026-09-09 (arXiv:2609.10416v1, cs.CR / cs.SE)
**Link:** https://arxiv.org/abs/2609.10416
**Code:** not stated

---

## Problem Statement (plain English)

When a coding agent produces a patch, you can watermark the *final artifact* to prove provenance — but that doesn't authenticate the *process* that produced it. Existing behavioral watermarking gives only a global "is this ours?" signal, so if someone locally edits a trajectory it may still look owned, and you can't tell **which protected region was tampered with**. The paper wants both: robust ownership attribution *and* fine-grained detection of where a trajectory was modified.

## Methodology (technical)

TrajMark is a **training-free, symmetric-key, visible-only** framework with two complementary layers:

- **Owner layer (robust, sparse):** encodes a **6-bit deployment identifier** by rewriting a keyed subset of naturally occurring **READ actions** into **masked linear equations**. Ownership evidence accumulates across the trajectory.
- **Localization layer (fragile, dense):** inserts linked **Q12 seals** — ordinary, group, and terminal — that commit to protected **critical-action segments**. Local edits perturb nearby keyed commitments and reveal the affected protocol region.
- Separating the two lets ownership stay robust while integrity stays sensitive.
- The authors provide a design-level analysis of owner recoverability, integrity collision probability, structural overhead, and localization behavior.

## Key Results (with numbers)

- Across **3 coding-agent frameworks and 3 LLMs**: recovers the **exact owner in 100%** of evaluated clean full-watermark batches.
- Under exhaustive eligible single-site attacks: **detects 95.5–100%** of edits.
- Under random single-action corruption: **localizes 95.8%** of modified sites to an accepted protocol region (region-level, not individual-action-level).
- **Owner marking adds no trajectory actions**; the integrity layer adds explicit read-only seals.
- Matched **Pass@1 = 26.9%** (utility preserved under the watermark).

## What's Novel

- Splits provenance into **robust ownership** vs. **fragile local integrity** — a deliberate separation that most single-signal watermarking schemes lack.
- Watermarks the **visible reasoning/action process**, not just the output patch, so the "how" is authenticated alongside the "what."
- **Training-free and visible-only**: no model retraining, no hidden/covert channel — matters for auditability and legal/policy acceptance.
- Provides region-level **tamper localization**, not just detection.

## My Connection

This is the provenance/attestation layer for agentic coding workflows Manny touches: it's a concrete mechanism to attribute agent trajectories and detect post-hoc tampering — directly relevant to supply-chain integrity for AI-generated code and to the "who actually wrote this patch" question that shows up in both red-team reporting and defensive forensics.

## What I Learned (plain English)

You can fingerprint the *process* a coding agent followed, not just its final diff — and you can do it without retraining anything. The clever move is using two watermarks on purpose: a sturdy one that survives edits to prove ownership, and a delicate one that *breaks* on edits to reveal where tampering happened. Ownership and integrity are different questions, so they get different tools.
