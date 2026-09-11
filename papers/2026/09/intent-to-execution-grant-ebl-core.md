# From Intent to Execution Grant: An Execution-Boundary Conformance Profile for High-Risk AI Actions (EBL-Core)

**Authors:** Mengting Wu, Lin Wang, Yong Zhang, Jiang Deng
**Institution:** (not stated in the arXiv listing)
**Published:** September 10, 2026
**Link:** https://arxiv.org/abs/2609.11596
**Artifact:** 28 pages, 2 figures, 8 tables, plus an ancillary minimal reference artifact (schemas, test vectors, executable validation)

---

## Problem Statement

AI agents increasingly propose actions with real-world consequences — financial transfers, infrastructure changes, software deployments, disclosures, physical actuation. Existing building blocks (authorization engines, policy languages, runtime monitors, provenance systems, agent guardrails) each cover part of the problem but share no common semantic contract for the single most dangerous moment: the transition from *a candidate action the model proposed* to *execution authority*. In plain English: everyone agrees the agent needs permission to act, but nobody has defined precisely what "permission to do this exact thing" means or how to verify it.

## Methodology

The paper specifies **EBL-Core**, an execution-boundary conformance profile whose job is to decide whether one canonical, fully materialized AI-generated candidate action may receive action-scoped execution authority under explicit conditions.

The design binds together, through an **Execution Release Contract (ERC)**:

- a structured **intent object**,
- **Root** and **Operational Policies**,
- **evidence obligations** and **typed evidence**,
- **context** and **time**,
- a **verifiable Decision Derivation** (so the decision is reproducible/checkable, not just asserted).

Key semantic choice: an ERC is explicitly **not** an authority-bearing token. A verified **ALLOW** ERC only *supports* a separate **Execution Grant**, which is itself governed by **redemption-time validation**. The profile also specifies action binding, **policy non-weakening** (a downstream layer may not loosen what upstream policy allowed), evidence handling, deterministic adjudication, derivation verification, and grant lifecycle behavior.

An accompanying reference artifact implements schemas, adjudication, separate verification, **Semantic Replay**, and a linearizable in-memory grant store.

## Key Results

- The retained run validated the profile against **34 static vectors** and **15 lifecycle checks**, all matching expectations (per the abstract).
- Formalizes the intent → ALLOW ERC → Execution Grant → redemption-validation pipeline as a *conformance profile* (a spec others can implement against), rather than a single tool.
- Provides an executable reference artifact so the profile can be tested, not just read.

## What's Novel

- Draws a hard line between **"the decision was allowed"** (ERC) and **"the action may now run"** (Execution Grant), with redemption-time validation closing the time-of-check/time-of-use gap.
- Treats **policy non-weakening** and **derivation verification** as first-class requirements — the decision must be both non-loosening downstream and independently re-checkable.
- Frames agent authorization as a **conformance profile** (comparable, testable, implementable) instead of another proprietary guardrail.

## My Connection

This is the authorization-layer sibling to the same-day "Whisper Attacks / AP2" paper. Whisper Attacks shows signatures validate the transaction but not the decision; EBL-Core specifies what *would* bind a decision to an execution grant. Together they define the semantic contract agent platforms (and Manny's red-team harnesses) are currently missing. It is also a natural fit for red-team tooling: the static vectors and lifecycle checks are the kind of artifact you point an adversarial fuzzer at to find where a supposedly conformant implementation honors the letter of the profile but not the intent.

## What I Learned

"Allowed" and "executed" are two different events with a dangerous gap between them — an AI can be granted permission and then have conditions change before the action fires. EBL-Core's answer is to make the grant redeemable only under re-checked conditions at the moment of execution, and to require that the decision be re-derivable rather than trusted on the say-so of whichever component made it.
