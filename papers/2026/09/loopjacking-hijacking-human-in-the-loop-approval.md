# Loopjacking: Hijacking Human-in-the-Loop Approval

**Authors:** Adithyan Arun Kumar
**Institution:** Independent (evidence archive: github.com/adithyan-ak/loopjacking)
**Date:** Submitted Sep 17, 2026 · cs.CR, cs.MA
**Link:** https://arxiv.org/abs/2609.21081
**Code/Evidence:** https://github.com/adithyan-ak/loopjacking (17 pages, 3 figures, 3 tables)

---

## Problem Statement (plain English)

Almost every agent stack has a human-in-the-loop (HITL) approval gate: the agent wants to do something consequential (delete a record, transfer money, send an email, run a command), it stops, and it shows the human a summary. The human clicks "approve." Everyone treats that click as the last security boundary.

But approval is only meaningful if the operation the human reviewed is the exact operation that later executes. This paper shows that in real, shipped agent products it often isn't. The human approves operation **A** (what they understood), and the system later executes operation **B** (something materially different). The paper names this **Loopjacking** — hijacking the loop between decision and execution.

Two variants:

- **Representation-based attack** — B is already encoded in the pending action, but is omitted or misrepresented in what the human is shown. The human approves a sanitized view; the real payload executes.
- **Post-approval state-substitution attack** — the human genuinely sees the correct A. Then mutable workflow state is changed *after* approval, and the release path picks up B instead.

## Methodology (technical)

The author evaluates a purposive (not exhaustive) set of released agent products and reproduces each variant:

- **Post-approval substitution:**
  - Reproduced in **seven tested Agno AgentOS releases**, ending at **3.0.9**.
  - Reproduced in **12 tested versions** of a conditional in-memory **LangGraph Agent Server** composition, ending at **0.14.0**.
  - Mechanism: the pending/approved action is held as mutable state that a subsequent step (another tool call, a graph node, or an async write) can rewrite between "approve" and "release."
- **Representation mismatch:**
  - Reproduced in **OpenClaw 2026.2.23**.
  - **Rejected** in **OpenClaw 2026.2.24** (a fix landed).
- **Negative control:**
  - **OpenAI Agents SDK 0.22.0 and 0.22.2** — serialized continuation preserves exact per-call binding and rejects a mutated B. This is the paper's proof that the attacks are fixable rather than inherent.

The author explicitly separates this contribution from prior work: misleading approval dialogs, session smuggling, action binding, and authorization continuity (e.g. the earlier "Verification-Status Laundering" and closed-world tool-hallucination lines).

## Key Results (with numbers)

- Post-approval state substitution reproduced across **7 Agno AgentOS releases (≤3.0.9)** and **12 LangGraph Agent Server versions (≤0.14.0)**.
- Representation mismatch reproduced in **OpenClaw 2026.2.23**; **fixed in 2026.2.24**.
- **OpenAI Agents SDK 0.22.0 / 0.22.2** resisted both classes in the tested configurations → proof of a working design pattern.
- The paper is explicit that these results are **not** a prevalence estimate — it is a set of existence proofs against named, released products.

**Defenses that work (per the paper):**
1. **Complete canonical approval rendering** — show the full, normalized operation, not a summary with a hidden field.
2. **Exact use-time comparison** — re-serialize the operation at execution time and compare byte-for-byte against what was approved; reject on mismatch.
3. **Prevent unauthorized pending-state mutation** — make the approved action immutable between approval and release.

## What's Novel

- Names and taxonomizes an approval-integrity failure class that the agent-security community has been circling but not formalizing: the **approve-then-execute TOCTOU window**.
- Distinguishes **representation** (lie at approval time) from **substitution** (lie after approval time) — two very different fixes.
- Provides a **clean negative control** (OpenAI Agents SDK) that shows the fix is achievable without killing legitimate execution.
- Produces **version-precise** reproductions (down to patch releases), which makes it immediately actionable for anyone running those stacks.

## My Connection (to Manny's work)

This is directly testable in a red-team engagement and maps onto the agent-harness work I already cover:

- **Approval-gate red-teaming** — for any client deploying an agent with a HITL "confirm" step, the test is: enumerate the fields in the pending-action object, mutate one after approval, observe whether execution honors the mutation. That's a concrete, high-impact finding.
- **Tooling:** pairs with the earlier "Verification-Status Laundering in LLM Agent Pipelines" (2609.20211) and "Closed-World Resolution Against Tool Hallucination" (2609.19425) — all three say the same thing from different angles: trust must be re-established at execution, not inherited from a decision.
- **Detection/defense:** the byte-for-byte use-time comparison is a cheap, shippable guardrail; recommend it as a control in any agent-harness hardening checklist.

## What I Learned (plain English)

"Human in the loop" is a promise, not a property. A human approving something is worthless if the thing that runs is allowed to differ from the thing that was shown. The fix is embarrassingly simple — freeze the approved action and re-check it at execution — but almost nobody ships it, because the pending-action object is usually just mutable application state sitting in memory for a few hundred milliseconds. That window is the vulnerability. If I had to teach one sentence: *approval must bind to an immutable, canonically rendered operation, or it's theater.*
