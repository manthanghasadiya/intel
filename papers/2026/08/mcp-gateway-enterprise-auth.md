# A Gateway Architecture for Enterprise MCP Authentication

**Paper:** [arXiv:2608.10760](https://arxiv.org/abs/2608.10760) — "A Gateway Architecture for Enterprise MCP Authentication: Unifying Heterogeneous Auth, Identity Delegation, and the User / Non-User Persona Problem"

**Authors:** Suraj Kumar, Amy Wang, Srinivasan Manoharan
**Institution:** Not stated on abstract page (industry deployment report)
**Published:** 2026-08-11 (cs.CR)
**Code:** Not listed on abstract page

---

## Problem Statement (plain English)

Enterprises went from zero to dozens of internally built MCP servers within a year. Every team implemented auth its own way — some servers have no auth at all, some use static API keys, some full OAuth. Result: there is no consistent way to (a) know who is allowed to call which tool, (b) audit what an agent actually did, or (c) revoke access when an employee leaves. The paper reports a production deployment that solves this with a centralized MCP gateway: one aggregation/governance/auth layer that fronts every downstream MCP server.

## Methodology (technical)

Four contributions, all grounded in a real deployment fronting dozens of MCP servers across web, desktop, custom-SDK, and low-code clients:

1. **Two-axis authentication model** — persona (interactive user vs. automated non-user) × credential type (no-auth, static/dynamic API key, PKCE, client credentials, platform app-context).
2. **Gateway auth layer** supporting three enterprise SSO grants and three token-provisioning models: Bring-Your-Own-Token, Generate-Your-Own-Token, and delegated OAuth via RFC 8693 token exchange.
3. **Three end-to-end identity flows** — User-to-OAuth2, Non-user-to-Service-Account, User-to-Service-Account — composing client, gateway, and downstream server.
4. **Deployment evolution** from CDN/WAF/edge perimeter to private MCP tunnels and enterprise-wide connectors.

## Key Results

- Architecture is **in production**, fronting dozens of MCP servers.
- Solves the fleet-wide offboarding problem: one place to revoke a departing employee's agent access instead of hunting per-team servers.
- Unifies the auth fragmentation: no-auth, API-key, and OAuth servers all sit behind one consistent authorization + audit layer.
- Handles the non-user persona explicitly (service accounts / automation), which per-team implementations almost always get wrong.

## What's Novel

Most MCP security work so far is offensive (prompt injection, malicious servers) or theoretical. This is a **working reference architecture** from an actual enterprise rollout — including the persona problem (interactive user vs. headless automation), which is the crux of MCP auth and rarely addressed. The RFC 8693 token-exchange delegation pattern is a concrete, standards-based answer to "how does the gateway act on my behalf without holding my password."

## My Connection (to Manny's work)

Every enterprise agent deployment Manny red-teams has this exact failure mode: MCP servers with inconsistent or missing auth, no centralized audit, no offboarding. The gateway pattern gives a concrete target architecture to test against — and a checklist of what to probe when it's absent (unauthenticated servers, static API keys in configs, service accounts without scoping, no per-request audit trail). Also: the Bring-Your-Own-Token model is itself a juicy attack surface (token reuse, delegation abuse).

## What I Learned (plain English)

The fix for "everyone built their own auth" isn't better per-server auth — it's putting one choke point in front of all servers and making every call pass through it. Persona matters as much as mechanism: a service account acting autonomously is a different security object than a human clicking in a UI, and auth designs that conflate them are how agents end up with human-level privileges. If you're deploying agents in an org, put a gateway in front of MCP before you have 20 servers to retrofit.
