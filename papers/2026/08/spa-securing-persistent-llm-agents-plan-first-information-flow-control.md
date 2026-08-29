# SPA: Securing Persistent LLM Agents Across Queries with Plan-First Information-Flow Control

- **Paper:** https://arxiv.org/abs/2608.27234
- **Authors:** Dylan Girrens, Guangjing Wang
- **Published:** 2026-08-27 (latest arXiv announcement; no new submissions Aug 28–29)
- **Code:** Not released (submitted to USENIX Security 2027; 24 pages, 17 figures)

## Problem Statement (plain English)

LLM agents read untrusted webpages, documents, and tools, and act on security-sensitive resources. Most defenses protect either the *planning step* or *individual tool calls*, but persistent agents run across many queries and keep state between them. That means attacker-controlled data can (a) change what the agent decides to do, (b) sneak into security-sensitive tool arguments, or (c) poison state that gets reused in a later query — including delayed attacks that fire queries later. How do you secure a long-lived agent across its whole lifetime, not just one turn?

## Methodology (technical)

SPA is a "plan-first" architecture with three layers:

1. **Plan-first planning:** The planner is invoked once per query and emits a *complete* executable plan in a declarative domain-specific language. Execution then follows that plan; the planner is not re-prompted mid-task with attacker-influenced context.
2. **Dual-lattice information-flow control (IFC):** Confidentiality and integrity labels are tracked across both explicit data flows *and* control dependencies. Untrusted data (e.g., web content) is labeled low-integrity; security-sensitive tool arguments require high-integrity data. Tainting propagates through control flow, not just data flow.
3. **Label-preserving persistence:** Execution results are stored as labeled artifacts. Later planning queries only see semantic *metadata* about those artifacts — the untrusted payloads themselves are never re-exposed to the planner, closing the cross-query poisoning channel.

Evaluation uses AgentDojo plus **AgentDojo-MQ**, a new multi-query extension built to measure secure state reuse and delayed attacks. The headline attack tested is `tool_knowledge` (attacker-controlled tool descriptions steering the agent).

## Key Results (with numbers)

- Under the `tool_knowledge` attack, SPA with IFC reduces attack success to **0% on AgentDojo** and **0.2% on AgentDojo-MQ** (down from high baseline success in prior work).
- Label-preserving persistence prevents delayed attacks where poisoned state would otherwise trigger in later queries.
- There is a measurable **security–utility tradeoff**: strict integrity enforcement blocks legitimate actions too — the paper is explicit that the cost exists and must be managed.

## What's Novel

- Most prior agent defenses are single-query or tool-level; SPA is end-to-end across **planning, execution, and cross-query state reuse**.
- **Persistence without re-exposure**: labeled artifacts + metadata-only replanning is a clean answer to the "poisoned memory" problem that memory-injection attacks (e.g., InjecMEM, 2608.23471) exploit.
- Plan-first + dual-lattice IFC with control-dependency tracking is a concrete, implementable architecture — not a prompt-level defense.

## My Connection (to Manny's work)

Manny builds and attacks agentic systems. This is the defensive counterpoint to the attack research in the last week's digests (instruction privilege escalation in harnesses, 2608.27299; memory injection, 2608.23471; framing-gap exfiltration, 2608.27092). SPA's design choices are exactly the ones those attacks fail against: no re-prompting mid-task, integrity labels on tool arguments, and metadata-only persistence. Worth evaluating SPA-style controls against Manny's own harness — and worth attacking: the security-utility tradeoff means real deployments will loosen labels, and label-lowering logic is a natural injection target.

## What I Learned (plain English)

- "Plan once, execute strictly" beats "re-plan every step with fresh context" — every re-prompt is a new injection surface.
- Integrity labels need to follow *control dependencies*, not just data dependencies, or tainted data sneaks in through decisions rather than values.
- The hard problem in persistent agents isn't one query — it's that state from query N gets reused in query N+1. Defenses must treat persistence as a first-class channel, and the only safe persistence is one that doesn't re-expose raw untrusted content to the planner.
- Any defense with a real utility cost will get configured down in production — which is where red teams should look.
