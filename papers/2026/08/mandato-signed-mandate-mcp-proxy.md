# Mandato: Protocol-Level Enforcement of Digitally Signed Mandates on AI Agent Actions

- **Link:** https://arxiv.org/abs/2608.14074
- **Authors:** Giovanni Racioppi
- **Institution:** Not stated in abstract (single author)
- **Published:** Aug 14, 2026 (announced in arXiv daily batch Aug 17)
- **Code:** Not linked in abstract (implementation status described in paper)

---

## Problem Statement (plain English)

When an AI agent calls tools through MCP, nothing at the protocol level stops it from doing something its operator never authorized. Authorization logic lives in application code: it's unsigned, not independently auditable, and the logs have no evidentiary value — if an agent exfiltrates data or destroys state, you can't cryptographically prove who authorized what. Mandato asks: what if every agent action had to carry a *digitally signed mandate* — like a power of attorney — and every decision (permit/deny) was recorded in a tamper-evident, court-usable audit trail?

## Methodology (technical)

A **governance proxy** sitting transparently in the MCP path:

1. **Mandate artifact:** a machine-readable, cryptographically signed authorization document specifying *which tools* an agent may invoke, *under which parameter constraints and contextual conditions*, *for how long*, and *on whose behalf*. Modeled deliberately on the civil-law institution of *delegation of authority* (mandato) so lawyers and auditors can read it, not just engineers.
2. **Decision semantics:** every tool call is evaluated against the applicable mandate chain — the proxy blocks non-conforming calls inline.
3. **Audit trail:** every permit/deny decision plus its evidence is recorded in an **append-only, hash-chained log**, periodically anchored with **qualified timestamps** for evidentiary use.
4. **Architecture:** separated decision and enforcement points, MCP-transparent (agents and MCP servers don't change).
5. **Regulatory mapping:** maps the mechanism onto EU AI Act Articles 12 and 14, GDPR accountability, NIS2, and eIDAS 2, including a roadmap to qualified attestation through Qualified Trust Service Providers (QTSPs).
6. **Evaluation plan (stated):** enforcement overhead, audit completeness, and tamper-evidence verification cost.

## Key Results (with numbers)

The abstract is a systems/positioning paper: it presents the mandate model, decision semantics, reference architecture, regulatory mapping, and implementation status, with a *quantitative evaluation plan* (enforcement overhead, audit completeness, tamper-evidence verification cost) rather than completed benchmarks. No headline numbers yet — the contribution is the protocol-level design and its legal-theoretical grounding.

## What's Novel

- Authorization as a **cryptographically signed, machine-readable artifact at the protocol layer** rather than ad-hoc checks in application code.
- **Hash-chained, timestamped audit logs designed for evidentiary use** — the logs are built to survive legal scrutiny, not just SIEM dashboards.
- **Civil-law framing (mandate/delegation)** — an explicit attempt to make agent authorization legible to lawyers and auditors, and to map it onto AI Act/GDPR/NIS2/eIDAS 2 obligations with a QTSP attestation path.
- MCP-transparency: retrofit into existing agent stacks without changing agents or servers.

## My Connection (to Manny's work)

This is directly adjacent to Manny's MCP security work (Customhouse, MCP gateway auth, agent exfiltration defenses): Mandato is the "signed authorization + evidence-grade logging" layer that provenance-based defenses like Customhouse currently approximate with policy logic. For Manny, the interesting moves are (a) the mandate-chain semantics for *nested* agents (sub-agent calls inheriting scoped authority), and (b) the attack surface of the proxy itself — a mandate parser and enforcement point is a prime prompt-injection/tool-confusion target, and its "block non-conforming calls" decisions are only as good as the constraint evaluator. Red-team angle: test whether a malicious MCP server can forge, replay, or escalate mandates.

## What I Learned (plain English)

- The industry's agent-authorization gap is now being framed as a *legal-evidence* problem, not just an engineering one — "who authorized this and can you prove it in court" is becoming a first-class requirement (EU AI Act, NIS2, eIDAS 2).
- Separating **decision** from **enforcement** is the standard governance architecture (same as policy engines in enterprise IAM) — and it's the right way to retrofit security onto protocols that were designed without it.
- Hash-chained audit logs + qualified timestamps is the pattern for making agent activity forensically usable; plain logs won't survive scrutiny.
- Mandates as "power of attorney for agents" is a genuinely useful mental model for scoping what an agent may do — including for Manny's own red-team agents, which should carry explicit signed scopes too.
