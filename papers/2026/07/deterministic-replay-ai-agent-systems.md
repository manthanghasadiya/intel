# Deterministic Replay for AI Agent Systems

**arXiv:** [2607.16200](https://arxiv.org/abs/2607.16200)
**Authors:** Rasheed Mudasiru
**Published:** July 21, 2026
**Code:** Released under MIT license (Go, single static binary)
**Tags:** #ai-agents #deterministic-replay #debugging #reproducibility

---

## Problem Statement (Plain English)

AI agents are non-deterministic — the same prompt can produce different results because of LLM sampling variance, API state changes, CDN header differences, and execution environment noise. This means you cannot replay an agent run to debug a bug or audit a security incident. If something goes wrong, you can't exactly reproduce it. Current observability platforms log what happened but can't replay it.

## Methodology

The author presents **agrepl**, a developer-first CLI framework:

1. **Record phase**: A MITM proxy intercepts all external interactions at the transport layer (HTTP/gRPC/WebSocket). Every request and response is serialised into a structured execution trace with timing, headers, and payloads.

2. **Replay phase**: Traces are replayed in a strictly isolated environment with zero outbound network access. The framework uses a request-key matching function K(s) to deterministically match replayed requests to recorded responses. A noise-aware diff algorithm classifies HTTP header divergence into signal tiers (meaningful) and noise tiers (irrelevant).

3. **Formal model**: Defines the agent execution model, proves the determinism invariant — that replay under the same trace produces identical agent behavior.

## Key Results

| Metric | Value |
|--------|-------|
| Replay fidelity (F) | 1.0 (perfect) |
| Median per-step latency reduction | 98.3% |
| Workloads tested | 5 (n=250 replay instances) |
| Implementation | Go, single static binary |
| License | MIT |

The 98.3% latency reduction comes from replaying from local traces instead of making real API calls.

## What's Novel

- First formal framework for deterministic replay of LLM agent executions at the transport layer
- Noise-aware diff algorithm that separates meaningful from irrelevant HTTP header divergence
- Proof of determinism invariant
- Practical implementation shipping as a single static binary with MIT license

## My Connection

For Manny's agent security work, agrepl provides the infrastructure to:
- **Forensic replay of security incidents**: When an agent is compromised, replay the exact execution to understand what happened
- **Red-team testing**: Replay adversarial prompts deterministically across different models to compare behavior
- **Audit trails**: A replayable trace is a stronger audit artifact than a log — you can verify exactly what the agent did
- **Regression testing**: After patching an agent security vulnerability, replay the original attack trace to confirm the fix

## What I Learned

Deterministic replay for AI agents is solvable at the transport layer with a MITM proxy approach. The 98.3% latency reduction during replay is huge — debugging becomes near-instantaneous instead of requiring real API calls. The noise-aware diff is a clever practical contribution: not all HTTP header differences matter for replay fidelity, and classifying them prevents false-positive replay failures.
