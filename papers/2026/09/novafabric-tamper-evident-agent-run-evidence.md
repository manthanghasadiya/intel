# NovaFabric: Tamper-Evident, Replayable Evidence for Autonomous AI Agent Runs

**Authors:** Mohsen Seyedkazemi Ardebili
**Institution:** (single-author; affiliation per arXiv listing)
**Published:** arXiv:2609.12582 — submitted 11 Sep 2026
**Link:** https://arxiv.org/abs/2609.12582
**Code:** Not specified in the abstract (artifacts referenced: 15-entity Run Capsule schema, four-mode replay protocol, Evidence Bundle export)

---

## Problem Statement (plain English)

When an autonomous AI agent does something consequential — moves money, changes infrastructure, reads sensitive data — what can you actually *prove* about what it did afterward? Today's agent-observability platforms capture "traces," but a trace is a mutable log: it can be altered undetected, it comes with no recipe for re-running the run to check it, and it says nothing about whether secrets were scrubbed before capture. Meanwhile regulation (EU AI Act, ISO 42001, NIST AI RMF) increasingly presumes records that an *independent party* can check. The gap: there is no portable, tamper-evident, replayable artifact that lets a third party verify an agent run without trusting the operator's own telemetry.

## Methodology (technical)

NovaFabric records an agent run **without modifying agent logic** into a portable **Run Capsule** — a 15-entity schema — sealed with a **holistic DSSE signature**, an **RFC 3161 timestamp**, a **Merkle log**, and a **redaction attestation** (a signed claim that declared secrets were removed). Sealed runs are **re-executable under a four-mode replay protocol** and exportable as an **Evidence Bundle** for third-party verification with stock tooling. The contribution is explicitly positioned as *integration, not new cryptography*: it composes **OpenTelemetry**, **DSSE/in-toto**, and **W3C PROV**. The paper evaluates eight research questions within a stated scope; verification is explicitly conditional on a stated trusted computing base (TBC).

## Key Results (with numbers)

- **Mocked replay** serves every model response from the capsule (no live model call): **10/10**. But it is offline **w.r.t. models, not the network** — only **2/10 tool-using workloads** completed; the identified gap is **missing tool-response substitution**.
- **Tampering** is rejected across **three tested classes**.
- **Declared-stream completeness:** **0.652** (95% CI ±0.064, ten scenarios).
- **Redaction:** a repaired rule pack redacts **14/14 credential types** while preserving **9/9 decoys**; a diff localises **140/140 mutations**.
- **Blast-radius queries:** **45.5 ms p99 over 10M edges** (3.3× faster than a columnar baseline); **167.9 ms over 100M edges** (1 client, n=30).
- **Distributed run:** a **314-machine, ten-region** run found capsule REST ingest lossless but **capped at 61.6 req/s (p99 26.8 s)** due to per-worker serialisation.
- **Self-audit:** six defects found in NovaFabric and its evaluation corpus — **four fixed, one withdrawn, one open**.

## What's Novel

Not the primitives — OpenTelemetry, DSSE/in-toto, and W3C PROV all predate this. Novel is the **composition into a single, portable, self-verifying artifact for agent runs** and, crucially, the **honest evaluation**: the paper reports where replay fails (tool-response substitution), quantifies completeness rather than claiming it, and publishes its own defect list (4 fixed, 1 withdrawn, 1 open). Refusing to claim full verification — and naming the trusted computing base — is the differentiator.

## My Connection (to Manny's work)

This is the concrete answer to the question "what artifact do you hand an auditor after a red-team run?" For agent-assurance content, NovaFabric gives a checklist to demand from vendors: sealed run capsule, Merkle-logged, DSSE-signed, timestamped, secret-redaction attestation, and a replay protocol that survives *tool* calls, not just model calls. The **2/10 tool-using replay** result is the headline weakness to probe: any "we can replay the attack" claim that doesn't substitute tool responses is theater. Directly usable as the evidence-backbone section of an Agent Assurance report.

## What I Learned (plain English)

Agent logs are not evidence. A log you control and can edit proves nothing to a regulator or an incident reviewer — tamper-evidence, timestamping, and *replayability* are what turn a trace into provable fact. But replaying an agent is much harder than replaying a model: you can fake the model's text responses, yet the moment the agent calls a real tool you have to fake the outside world too, and that's where even a careful design only completed 2 of 10 workloads. The lesson for anyone selling agent assurance: promise *auditable, tamper-evident* traces (achievable now) and be honest that full deterministic tool replay is still an open problem.
