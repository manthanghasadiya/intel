# Correct Is Not Governed: Provenance Integrity in Agentic Workflows

- **Paper:** [arXiv:2608.12761](https://arxiv.org/abs/2608.12761)
- **Authors:** Jesus Salas
- **Institution:** Not stated on arXiv listing
- **Published:** 2026-08-13
- **Code:** Not listed

## Problem Statement (plain English)

Most agent evaluations ask one question: did the workflow produce the right outcome? In institutional settings that's not enough. A workflow can reach a correct-looking result while acting on the *wrong authority* (a low-privilege tool claiming admin approval), *claiming completion it can't support* (saying "done" without evidence), or *working from stale facts* (a decision based on data changed after the fact). This paper argues for "governed execution": work whose decisions, completion claims, and responses to change are backed by inspectable provenance — and shows what enforcing that costs.

## Methodology (technical)

- **Matrix:** a deterministic causal-state layer interposed on agent workflows that:
  - records **authority dependencies** (who/what authorized each decision) and **fact dependencies** (which facts each result rests on),
  - **verifies completion evidence** before allowing a workflow to declare success,
  - **selectively invalidates** dependent work when an upstream fact or authority changes.
- **Evaluation 1 — controlled comparison:** governed vs. direct (ungoverned) workflows across the same tasks.
- **Evaluation 2 — role-separated transfer challenge:** a deterministically enforced completeness contract is applied to synthetic packets produced *outside* its authoring context.

## Key Results (with numbers)

- Governed and direct workflows **often reached the same outcomes**, but only the governed path consistently preserved governing evidence, refused unsupported closure, and limited recovery to dependent tasks.
- **Transfer failure:** the deterministically enforced completeness contract **severely over-blocked synthetic packets** produced outside its authoring context — governance that's rigid across contexts rejects legitimate work.
- The author is explicit: Matrix is not a general accuracy enhancer; its primary role is an *institutional integrity layer*.

## What's Novel

Reframes agent correctness from outcome to **process integrity**: provenance (authority + facts + completion evidence) as a first-class evaluation criterion, plus selective invalidation of dependent work when the world changes. The over-blocking result is a rare published measurement of the governance-vs-throughput tradeoff — hard guarantees cost legitimate flexibility.

## My Connection (Manny's work)

When Manny evaluates agent pipelines for enterprise deployment, "it worked" is the wrong bar — the question is whether the agent can *prove* it had the right authority, current facts, and real completion evidence. The over-blocking finding is also a red-team lever: completeness/authority contracts are exploitable in both directions (forged evidence to pass, or rigid contracts to DoS legitimate workflows).

## What I Learned (plain English)

A correct answer with no provenance is not a governed answer — and in audit-heavy environments, ungoverned correctness is nearly as bad as failure. But provenance enforcement isn't free: contracts strict enough to block unsupported claims will also block legitimate work they weren't designed for. Governance is a design tradeoff, not a checkbox.
