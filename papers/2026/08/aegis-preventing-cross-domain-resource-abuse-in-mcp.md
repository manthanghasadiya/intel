# AEGIS: Preventing Cross-Domain Resource Abuse in MCP

- **Link:** https://arxiv.org/abs/2608.20481
- **Authors:** Shriti Priya, Teryl Taylor, Frederico Araujo
- **Institution:** IBM Research
- **Published:** Aug 24, 2026 (submitted Aug 20, 2026)
- **Code:** Not released (per abstract page)

## Problem Statement (plain English)

MCP (Model Context Protocol) is the standard way LLM agents call external tools. Attackers or malicious agents can abuse *how* tools are invoked — not to steal data, but to degrade or break the service. Examples: an agent requests an absurdly large search radius, or very long videos, overloading backend systems into slowdowns or denial-of-service. Each input modality (text, images, video, location) opens a different abuse vector, and because tools span domains with wildly different request schemas, nobody has a consistent way to write mitigation policies that are both general enough and precise enough to enforce.

## Methodology (technical)

- Systematic analysis of MCP tool modalities (text, image, video, location) as distinct resource-abuse surfaces.
- Identifies why existing mitigation fails: multimodal + cross-domain tools expose heterogeneous schemas/parameters, so a single policy language either over- or under-fits.
- Proposes AEGIS: a policy framework for detecting and preventing cross-domain resource abuse in MCP-based agent applications, designed to be generalizable across tool domains while precise enough to enforce meaningfully (per the abstract's framing; full details are in the paper body).

## Key Results (with numbers)

- The abstract confirms the abuse classes and the mitigation gap but does not headline specific numeric results in the available text — the contribution is the taxonomy of modality-specific abuse vectors plus the AEGIS policy framework for MCP.

## What's Novel

- First paper explicitly targeting *resource abuse* (availability/DoS) as an MCP attack class, distinct from the well-trodden prompt-injection/exfiltration angle.
- Treats modality (text/image/video/location) as the organizing axis for abuse and defense, rather than per-tool fixes.

## My Connection (to Manny's work)

Directly extends the MCP pentest checklist: AEGIS's modality-abuse model is a concrete test case for agent deployments — try requesting max-radius searches, huge media payloads, and cross-domain chained tool calls to see if resource limits are enforced. Pairs with the Claws-in-Plain-Sight tool-call argument leak as a combined "MCP abuse" audit module.

## What I Learned (plain English)

MCP abuse isn't only about tricking the model — it's about weaponizing the *tool contract itself*. If an MCP server exposes a tool with unbounded parameters, a hostile or compromised agent can turn a normal API call into a DoS primitive, and fixing it per-tool doesn't scale. Policy needs to be cross-domain and modality-aware from day one.
