# Fresh Memory, Stale Plans: Dependency-Scoped Validation for Distributed LLM-Agent Memory

- **Paper:** [arXiv:2609.03340](https://arxiv.org/abs/2609.03340) — *Fresh Memory, Stale Plans: Dependency-Scoped Validation for Distributed LLM-Agent Memory*
- **Authors:** Evan Chen, Shiqiang Wang, Christopher G. Brinton
- **Published:** September 3, 2026 (cs.AI)
- **Code:** Not specified in coverage (community write-ups name the approach "PlanFence")

---

## Problem Statement (plain English)

Distributed LLM-agent teams suffer from a paradoxical failure: an agent reads the newest shared state — memory is fresh, records are current — and then acts on an **obsolete plan** anyway. The plan was built from older dependencies, and nothing re-validates it against the new facts before the agent acts. The paper names and measures this "stale-plan execution" failure class and proposes a fix.

## Methodology (technical)

- Identifies stale-plan execution as a distinct failure mode in distributed agent memory systems: freshness of *facts* (memory reads) is decoupled from freshness of *plans* (derived artifacts with their own dependency graphs).
- Proposes **dependency-scoped validation**: when any dependency feeding a plan changes, only the affected plan components are flagged for re-validation, rather than re-planning wholesale.
- Evaluation uses **controlled replay**: fixing the decisions in **30 public workflow templates** while replaying identical memory, transport, and validation paths through **3–8 agent-local copies**, enabling policy comparison under identical conditions.

## Key Results (with numbers)

- Demonstrates that fresh memory alone does not prevent stale-plan execution — agents act on outdated plans despite reading current facts.
- Dependency-scoped validation (PlanFence-style) catches plan/dependency skew at the component level, avoiding full re-planning cost.
- Controlled-replay methodology (30 templates × 3–8 copies) provides a reproducible comparison baseline for memory/validation policies.

## What's Novel

Prior work focused on memory *staleness* (agents reading outdated data). This paper isolates the complementary failure — **plans going stale while memory stays fresh** — and treats plan validity as dependency-scoped state that needs its own validation lifecycle, plus a rigorous replay harness for testing validation policies.

## My Connection (to Manny's work)

This is a ready-made red-team primitive. A plan-integrity attack: update the shared knowledge base mid-task (or let a second agent do it) and observe whether the planning agent executes its obsolete plan. For Manny, that's a testable vulnerability class across agent harnesses (Codex CLI teams, Claude Code multi-agent, MCP-backed planners) — and the paper's controlled-replay setup is essentially a test harness blueprint.

## What I Learned (plain English)

"Fresh data" and "fresh plan" are two different things. An agent can have perfect, up-to-date information and still march forward on yesterday's plan — so systems need to check whether the *plan's inputs* changed, not just whether the agent can read new facts. Any distributed agent setup should re-validate plans against their dependencies before high-impact actions, and red teams should probe exactly that gap.
