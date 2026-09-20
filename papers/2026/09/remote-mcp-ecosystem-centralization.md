# Characterizing Network Centralization and Observability in the Remote MCP Ecosystem

**Author:** Muhammad Abdullah Sohail
**Published:** 2026-09-16 (arXiv v1)
**Link:** https://arxiv.org/abs/2609.19100
**Code:** Not stated in abstract

---

## Problem Statement (plain English)

MCP is now the default way agents reach external data and execution environments, and the ecosystem has quietly shifted from *local* process execution to *remote* Streamable HTTP deployments. That shift introduces scale-level architectural and security constraints nobody has actually measured. The paper asks basic questions the field has been guessing at: How concentrated are remote MCP servers? Who actually enforces authentication — the individual operator or the hosting platform? And can a security team even *scan* the ecosystem for tool-poisoning without first getting credentials?

## Methodology (technical)

A **three-tier observability framework** applied to real remote MCP endpoints:

- **O₀ — Catalog metadata:** what registries publicly declare.
- **O₁ — Passive compliance signals:** gateway/auth behavior observable without credentials.
- **O₂ — Live vulnerability analysis:** active probing for tool-poisoning vectors.

Applied to a **stratified sample of 179 remote endpoints** drawn from **two primary public registries**, with concentration measured via the **Herfindahl-Hirschman Index (HHI)** computed over the **Autonomous System Number (ASN)** distribution and auth mechanisms correlated against **hosting platform class** (e.g. commercial PaaS vs self-hosted).

## Key Results (with numbers)

- **HHI = 0.736** over ASN distribution — far above the **0.25** threshold for a "highly concentrated market." Remote MCP hosting is an oligopoly, not a long tail.
- **95% of commercial PaaS-hosted servers enforce gateway-level OAuth 2.1 with PKCE** — i.e. authentication is a property of the *platform*, not the individual server operator's configuration.
- **Security–Observability Tradeoff:** the very platform-level auth that secures the majority of servers also **blocks automated vulnerability scanning**. Gateway operators cannot assess tool-poisoning vectors without prior credential provisioning — security and measurement are in direct tension.
- The framework generalizes beyond the sample: catalog → passive → live is a repeatable pipeline for ecosystem-wide MCP measurement.

## What's Novel

1. **First measured concentration metric for the remote MCP ecosystem** — turning "MCP is consolidating" from anecdote into an HHI number.
2. **Shows auth is inherited from infrastructure, not chosen by operators.** That has a big implication: fixing MCP security at scale means fixing *PaaS gateways*, not writing guidance for thousands of individual server authors.
3. **Names the Security–Observability Tradeoff explicitly.** This is the paper's sharpest contribution — well-intentioned auth deployment directly degrades the ecosystem's ability to audit itself.
4. **Provides a three-tier framework that distinguishes what you can learn without credentials from what requires them** — practically useful for anyone building MCP inventory/recon tooling.

## My Connection (to Manny's work)

This is a **recon-and-reality-check** paper for MCP risk, and it's directly actionable:
- **Content angle:** "The MCP ecosystem is as concentrated as the US airline industry — and auth is the reason you can't scan it." HHI 0.736 is a quotable, verifiable anchor.
- **Red-team angle:** it tells you where the *real* trust boundaries are. If 95% of PaaS-hosted servers ride the same gateway OAuth pattern, a flaw in that gateway class is a mass-compromise primitive — that's where offensive attention should go, not at individual tool authors.
- **Defense/library angle:** the O₀/O₁/O₂ tiers are a clean model for how Manny frames MCP inventory methodology: what's free, what's passive, and what needs creds.
- Companion read to the same author's NIDS-evasion paper (2609.19091) — concentration *and* invisibility in one ecosystem.

## What I Learned (plain English)

Ecosystem health and ecosystem security can pull in opposite directions. MCP is quietly consolidating onto a handful of networks and hosting platforms, and the platform-level login protections that make it safer are the exact same thing that makes it impossible to audit. So "just add auth" isn't a fix — it's a trade. And when risk concentrates that hard, you stop auditing thousands of little servers and start staring at the few big gateways that hold them all up.
