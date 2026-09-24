# Agent Name Collision Attacks in Multi-Agent Systems

**Authors:** Adithyan Arun Kumar | **Published:** Sep 23, 2026
**Link:** https://arxiv.org/abs/2609.27624
**Code:** Not stated in the listing (regression tests run against seven pinned open-source revisions)
**arXiv ID:** 2609.27624v1

---

## Problem Statement

Multi-agent hosts turn **remote Agent Cards** into local agents, tools, workflow targets and broker routes. The problem is that the **A2A protocol defines a card's `name` as human-readable metadata — not a stable identity** — and specifies **no collision semantics**. The security failure begins the moment a host nevertheless uses that remote name as a **local routing identifier**: two peers can claim the same name, and a request addressed to a trusted peer's name can be dispatched to an attacker-controlled peer.

## Methodology (technical)

- Traced the full path from **registration through dispatch**, and ran **isolated regression tests at seven pinned open-source revisions**.
- Two test styles: **client-style integrations** (which directly select a peer endpoint) and a **brokered implementation** (which derives routes from names).
- Used **synthetic credential and tool tests** to measure whether a collision actually transfers identity or tool authority between peers, rather than assuming it does.

## Key Results (with numbers)

- **Six** client-style integrations selected an **attacker-controlled peer's client or loopback endpoint** for a request addressed to a **trusted peer's name**.
- A **seventh**, brokered implementation **collapsed both peers onto one name-derived route**; queue and access-control state determine whether the outcome is **interception or denial**.
- The common result is **wrong-peer dispatch — not universal privilege inheritance**.
- Synthetic tests found **no A-specific credential transfer** in the tested client bindings and **no direct transfer of A-owned tools**.
- The broker path forwards a **caller-configuration object**; delegated identity or tokens reach peer B only if present *and* B can consume the route. Two further paths expose a **model-mediated decision** rather than direct execution authority.

## What's Novel

- Establishes a **recurring implementation vulnerability class** in agent-to-agent routing (the "name collision"), cleanly separated from protocol-level speculation.
- Rigorously bounds impact: it's **wrong-peer dispatch**, and the paper is careful to *not* overclaim a universal A2A protocol exploit or count of vulnerable deployments.
- Assigns distinct responsibilities across **protocol, implementations, and deployments** rather than blaming one layer.

## My Connection (to Manny's work)

Naming and routing are exactly the plumbing Manny wires when connecting MCP servers and A2A agents. The hard rule this paper supports: **route by an origin-bound stable identity, keep names presentational only, and reject ambiguous aliases** at registration. Worth adding a name-collision regression test to any agent gateway/registry Manny runs.

## What I Learned (plain English)

In agent systems, a "name" is not an identity. If your host routes messages by the name a remote agent *calls itself*, whoever claims that name can intercept requests meant for someone else. The fix is to route by something origin-bound and unforgeable (keys/origin), treat display names as cosmetic, and refuse duplicates instead of picking one silently.
