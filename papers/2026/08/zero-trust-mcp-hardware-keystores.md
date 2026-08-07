# Hardware Keystores for AI Agent Signing Workflows: A Zero-Trust MCP Enforcement Architecture

- **Paper:** https://arxiv.org/abs/2608.06130
- **Authors:** Leo Sambrook, Sampo Sovio
- **Institution:** Not fully listed in arXiv metadata (Sovio is affiliated with Aalto University)
- **Published:** 2026-08-06
- **Code:** https://anonymous.4open.science/r/Hardware-Keystores-for-AI-Agent-Signing-Workflows-Artifact-357C (anonymous artifact)

---

## Problem Statement (plain English)

AI agents constantly do cryptographic operations — signing Git commits, authenticating API calls, issuing certificates. Today their private keys live in software-accessible places: plaintext files, environment variables, container memory. Any process with read privileges can lift the raw key. The paper opens with a real production incident: keys were **exfiltrated from a widely deployed agent framework via email injection in under five minutes**. Two requirements are needed: (1) key *confidentiality* — the key material must never be readable by software, and (2) *content-aware authorization* — the key should only be used for operations the agent is actually authorized to sign, not whatever a prompt injection asks for.

## Methodology (technical)

- **Hardware confinement:** software-resident keys are replaced with hardware-confined keys (HSM, TPM, smart card) accessed through a vendor-neutral **PKCS#11** interface. The hardware device performs the crypto operation; the host only ever sees opaque handles and results — raw key material never enters memory.
- **Five-layer Zero-Trust enforcement stack** around the hardware boundary:
  1. **Session identity (SAGA)** — binds key use to a specific agent session,
  2. **Scope bounds (Smax)** — caps what operations/key uses are permitted,
  3. **Semantic validation (RAV)** — validates the *meaning* of what's being signed,
  4. **Taint tracking** — traces untrusted data through the pipeline,
  5. **Hardware execution boundary** — the final enforcement point.
- **Evaluation:** 12 injection scenarios derived from AgentDojo's ImportantInstructionsAttack template (Debenedetti et al., arXiv:2406.13352), run across four LLM models (gpt-oss-120b, Qwen2.5-72B, DeepSeek-V4-Flash, plus one more; n=192 combined).

## Key Results (with numbers)

- **Baseline Attack Success Rate (ASR): 19.3%** (95% CI [14.3%, 25.4%]) — three of four models follow injections in the unprotected baseline.
- **Protected ASR: 0%** (Wilson 95% CI upper bound 2.0%) — the zero-trust stack eliminates the attack class in this test set.

## What's Novel

Most agent security stops at prompt filtering or sandboxing — this is a **hardware root of trust for agent key use**, and it solves both halves: confidentiality (keys never in software) *and* content-aware authorization (the HSM won't sign what the semantic layer rejects). The vendor-neutral PKCS#11 interface means it's not locked to one TPM vendor, which makes it deployable in the heterogeneous MCP/agent world.

## My Connection (to Manny's work)

Manny works MCP security daily — MCP servers are exactly where agents do signing, auth, and tool calls with keys sitting in env vars. This architecture is a concrete blueprint for hardening MCP signing workflows: put keys behind hardware, add semantic validation before the hardware signs. The 19.3% → 0% ASR number is a strong, citable argument for hardware-backed agent key management in any client's MCP deployment.

## What I Learned (plain English)

- Software-resident keys are a solved-problem-adjacent disaster: one prompt injection exfiltrated keys in under five minutes in production.
- Hardware confinement alone isn't enough — the authorization layer (what gets signed) matters as much as key secrecy. The five-layer stack is the "zero trust" part; the HSM is just the enforcement point.
- 0% ASR with a 2.0% upper bound is the kind of result that survives scrutiny — worth benchmarking against Manny's own MCP test harness.
