# CONTINUITY: Security-Context Contracts for Composable LLM Agent Controls

- **Link:** https://arxiv.org/abs/2609.05269
- **Authors:** Chris Zheng, Geng Yang
- **Institution:** — (not stated in abstract)
- **Published:** Sept 4, 2026 (announced Sept 7, 2026)
- **Code:** Not mentioned in abstract

## Problem Statement (plain English)

Modern LLM agent stacks bolt together many security mechanisms — provenance tracking, authorization checks, policy engines, protocol adapters, execution sandboxes. Each one can be perfectly correct *on its own*, but when an action flows across component boundaries, security-critical context gets dropped, widened, rebound, or reinterpreted. The paper calls this **security-context discontinuity**: the end-to-end system is insecure even though no individual component is broken. There was no formal way to prove that a chain of agent controls actually preserves its guarantees from instruction to external effect.

## Methodology (technical)

CONTINUITY models every security component with an **assume-guarantee contract** and carries authenticated security context across transitions using six primitives:

1. **Signed root grants** — authenticate the origin of authority.
2. **Provenance commitments** — bind actions to their originating task/data.
3. **Role-bound transition receipts** — record who/what authorized each hop.
4. **Bounded typed releases** — limit what context may be widened and how.
5. **Transformation witnesses** — certify context transformations.
6. **Effect-bound execution permits** — scope what an execution may actually do.

It formalizes **consequence integrity**: every realized external effect must be backed by a valid, current authorization witness linking principal, task, provenance, delegation, policy state, canonical action, and finality boundary. The authors implement a reference verifier and a deterministic cross-layer fault-injection suite covering 32 fault classes across four application domains.

## Key Results (with numbers)

- **2,560 parameterized attack instances** spanning **128 fault-domain classes**.
- Full CONTINUITY configuration: **zero harmful external effects** committed.
- Completed all **700 benign tasks** (no availability loss from the controls).
- Escalated all **200 ambiguous cases** (presumably to human review) without harmful effect.

## What's Novel

Moves agent security from "each control is sound" to "the *composition* is sound." Treats the instruction-to-effect path as the security boundary and gives it a verifiable contract formalism — closer to what hardware/OS vendors do with attestation and capability models than typical agent-guardrail papers.

## My Connection (to Manny's work)

Directly applicable to how Manny audits agent harnesses: the vulnerability class here (context dropped/rebound across component hops) is exactly what real agent exploits — MCP tool chaining, hook bypasses, adapter confusion — tend to abuse. The contract primitives are a usable checklist for red-team findings ("this delegation hop loses provenance," "this adapter widens scope"), and the verifier/fault-injection design is a model for automated agent-security test suites.

## What I Learned (plain English)

"You have to look at the seams." Securing an agent is not securing the model, the tools, or the policy engine — it's proving that security meaning survives every hand-off between them. When auditing, trace the *context*, not the code path: who authorized this effect, and did that authorization survive each component boundary intact?
