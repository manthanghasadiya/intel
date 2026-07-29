# Paper Breakdown: The Missing Layer — Specification Infrastructure for AI Oversight

- **arXiv:** [2607.24866](https://arxiv.org/abs/2607.24866)
- **Date:** July 29, 2026
- **Authors:** Satyam Kumar, Saurabh Jha
- **Categories:** cs.CR
- **Code:** CARMA prototype referenced

---

## Problem Statement

Every team deploying an agentic system builds its own audit schema, policy dialect, monitoring stack, and escalation path — mostly reinventing patterns others have already figured out. This is a coordination gap, not a research gap. The missing piece is a shared "specification layer" that translates human intent into machine-checkable artifacts.

## Methodology

1. Two-axis taxonomy: **5 technical layers** (Legibility, Specification, Mediation, Evaluation, Escalation) × **6 concerns** (Alignment, Robustness, Adversarial Defense, Security, Governance, Accountability).
2. Populated the resulting 5×6 matrix with existing work to identify gaps.
3. Identified Layer 2 (Specification) as the connective tissue that every other layer depends on — yet lacks shared vocabulary, design principles, composability standards, and governance practices.
4. Proposed 6 design principles for Layer 2: elicitability, composability, adversary-awareness, traceability, governability, (plus one more).
5. Built CARMA (a Layer 2 prototype) for autonomous ETL agents where one specification drives enforcement, evaluation, and escalation.

## Key Results

- Existing systems (Cedar, Constitutional AI, Open Policy Agent) each address a fragment of Layer 2 — and the matrix — poorly.
- The proposed design principles make composition tractable across independent teams.
- CARMA prototype demonstrates that every decision can be traced to a versioned specification.

## What's Novel

The framing that AI oversight's core problem is a missing engineering discipline (specification infrastructure), not a missing research result. The 5×6 taxonomy is a practical tool for any team building agent governance.

## Connection to Manny's Work

Directly relevant to MCP security policy, agent authorization, and audit infrastructure. The Layer 2 (Specification) gap is exactly the problem Manny hits when defining what MCP servers an agent can call, under what conditions, with what escalation path. CARMA's "one specification drives enforcement, evaluation, and escalation" pattern is a reference architecture for MCP agent governance.

## What I Learned

The distinction between "research gap" and "coordination gap" is crucial. Most of what we need for safe AI agents already exists in fragments — the bottleneck is getting those fragments to compose. The 5×6 matrix is a useful diagnostic: for any agent deployment, map where you are on each cell to find blind spots.
