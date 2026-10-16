# My Credentials, Please: Detecting Filesystem Capability Laundering in MCP Servers at Scale

**Authors:** Zeqian Huang, Mingyue Tang, Aonan Guan, Qiming Sun, Pengzhou He
**Institution:** (multi-author, cs.CR)
**Date:** Submitted Oct 15, 2026
**Link:** https://arxiv.org/abs/2610.22301
**Code:** Not stated in abstract (paper references MCP-FSD tooling)

---

## Problem Statement (plain English)

MCP servers advertise each tool with a contract: a name, a description, and an argument schema. But when a client (agent) approves a call, it approves the *whole server process*, not just the declared operation. That process typically runs with broad host filesystem permissions. So an attacker who can steer the tool's arguments can make the tool read sensitive files on the host that are nowhere near its declared purpose. The paper names this gap **filesystem capability laundering**: a tool declared as non-filesystem (say, "fetch a URL") is used to slurp credentials off disk.

## Methodology (technical)

The authors analyse the capability gap at three layers:
- **L1 — server-category screening:** does the server's functional classification match the filesystem capabilities it actually exercises?
- **L2 — per-tool contract analysis:** compare each tool's declared argument schema/scope against the file paths it can actually reach.
- **L3 — path-validation checks:** test whether the server's own path-validation guards can be bypassed.

They build **MCP-FSD**, a pipeline that:
1. catalogs MCP servers,
2. infers a least-privilege boundary per tool,
3. plants credential fixtures *outside* those inferred boundaries,
4. runs **paired dynamic execution** to measure both exploitability and whether sandbox policies actually contain the access.

## Key Results (with numbers)

- Catalogued **21,097 MCP servers** across **nine registries**.
- Of **4,806 triaged servers**, **787 (16.4%)** exhibited filesystem capabilities despite being classed as non-filesystem.
- Dynamic three-case validation confirmed **150 vulnerable instances** under tested sandbox policies.
- Coordinated disclosure produced **13 security advisories**, including **two CVEs in Anthropic's Git MCP Server**.

## What's Novel

- First scale measurement of *capability laundering* (as opposed to classic path-traversal or command-injection bugs) in MCP servers.
- The insight that approval is granted at process scope, not operation scope, and that this is measurable across a whole ecosystem.
- A three-layer taxonomy plus a paired dynamic-execution harness that separates "exploitable" from "contained by sandbox."

## My Connection (to Manny's work)

This is square in Manny's wheelhouse: least-privilege tool scoping for agents and MCP servers. The 16.4% figure and the two Git MCP Server CVEs give concrete, citable evidence for the argument that MCP tool *descriptions* are not security boundaries. The MCP-FSD method (plant fixtures outside the declared scope, then execute) is a reusable red-team recipe Manny can adapt into an MCP audit lab.

## What I Learned (plain English)

Approving an MCP tool call is like handing a contractor the keys to your whole house because they said they'd only fix the sink. The fix isn't better prompts — it's enforcing the declared contract at runtime (sandbox the tool to exactly the paths it's allowed to touch) and treating every server as untrusted by default. Half of the ecosystem still fails the "does the declared scope match the real scope" test.
