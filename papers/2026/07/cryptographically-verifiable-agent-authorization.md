# Toward Cryptographically Verifiable Authorization for Autonomous AI Agents

**Authors:** M. Llambí-Morillas, D. Fernández-Fernández
**Link:** https://arxiv.org/abs/2607.21325
**Published:** 2026-07-23
**Submitted to:** ACM Transactions on AI Security and Privacy (TAISAP)
**Code:** Not specified in abstract (zk-SNARK PoC using Groth16)

---

## Problem Statement (Plain English)

When an AI agent acts on your behalf — reading a file, calling an API, spending money — how do you know it was *really* authorized to do that? Current systems use passwords or API keys, but those just prove "someone logged in." They don't give you a cryptographic receipt that says "Agent X was specifically allowed to take Action Y under Policy Z in Context W." This paper argues agent authorization should produce cryptographic proof — a zero-knowledge receipt — that binds the agent, the request, the policy, and the context together, without leaking private details.

## Methodology (Technical)

1. **Formal model (CVA):** Defines a relation *R_CVA* that jointly binds: an agent principal (`p`), a concrete authorization request (`req`), an execution context (`ctx`), and satisfaction of an applicable policy (`π`). Produces a proof `π_CVA` that any verifier can check.

2. **Security properties defined:**
   - **Authorization soundness:** A valid proof guarantees the agent was genuinely authorized.
   - **Principal binding:** The proof is cryptographically tied to a specific agent identity.
   - **Request binding:** The proof commits to the exact action requested (no scope creep).
   - **Policy binding:** Which policy was applied is part of the proof.
   - **Replay resistance:** The proof cannot be reused for a different request or context.

3. **Implementation:** Groth16 zk-SNARK (zero-knowledge Succinct Non-interactive ARgument of Knowledge) — an efficient proof system that lets a prover (the agent) convince a verifier that the authorization relation holds, while keeping private attributes (like specific policy rules) confidential.

## Key Results (with numbers)

- The paper is primarily theoretical — formal model + security property definitions.
- PoC demonstrates that the CVA relation can be instantiated over a Groth16 zk-SNARK construction.
- Identifies the "structural separation problem" as a central open challenge: identity binding, authorization-request binding, and runtime-execution binding are conceptually distinct but current frameworks treat them as interchangeable.
- No benchmark numbers, no attack success rates — this is foundations work.

## What's Novel

- First formal treatment of *cryptographically verifiable* (not just administratively verifiable) authorization for autonomous AI agents.
- Explicitly distinguishes identity binding from authorization-request binding from runtime-execution binding — current frameworks (OAuth, OIDC, SPIFFE/SPIRE, MCP auth schemes) conflate these.
- Proposes zero-knowledge proofs as the mechanism to preserve authorization attribute confidentiality while maintaining verifiability.

## My Connection (to Manny's Work)

Manny works on MCP security and agent authorization. The structural separation problem this paper identifies — that current auth frameworks don't distinguish *who* the agent is from *what* it's allowed to do from *when/where* it's doing it — maps directly onto MCP's current trust model challenges. The paper's "pre-execution authorization" framing is the missing layer between MCP service discovery (who to talk to) and runtime policy enforcement (what's allowed).

## What I Learned (Plain English)

- Current agent auth mechanisms (API keys, OAuth tokens, SPIFFE identities) are **not cryptographic authorization** — they prove identity or delegation, but they don't prove that a specific action was *permissibly* taken in a specific context.
- The "structural separation" insight is subtle but important: you can have perfect identity binding (you ARE Agent A) and still have no idea whether Agent A was actually *allowed* to perform the action it just took.
- Zero-knowledge proofs let you prove authorization without revealing the policy rules themselves — useful when the policy contains sensitive business logic.
- This is early-stage work (submitted to ACM TAISAP, preprint), but the formal model gives language to talk about MCP authorization gaps precisely.
