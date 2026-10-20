# Connection Escrow: Holding AI-Agent Pod Egress in the Kernel Until an Isolated Verifier Rules

**Authors:** Ashok Subbabhatta Gopalakrishna (single author) | **Affiliation:** not listed on the arXiv record
**Published:** October 19, 2026 | **arXiv:** [2610.26501](https://arxiv.org/abs/2610.26501) (cs.CR, cross-listed cs.NI, cs.OS)
**Code:** [github.com/45h0kg/benign-attach-gateway](https://github.com/45h0kg/benign-attach-gateway) | **Length:** 12 pages, 4 figures, 6 tables

---

## Problem Statement

Every existing control between an AI agent and the network lives at the wrong layer. A sidecar proxy, an application-level allowlist, or a tool wrapper all sit *above* the socket — so a child process that opens its own socket, a renamed binary, or a raw `connect()` walks straight past them. Worse, when a policy layer "approves" an action, that approval is a string in a log; it is not bound to the specific kernel socket that the agent later opens. Approve a request, and the agent is free to open forty more.

The result is that agent egress control is advisory. You can observe, you can alert, you cannot actually *prevent* the first byte from leaving.

## Methodology

**The hold point.** A 98-instruction eBPF program attached to the pod's cgroup hooks `connect()` and parks the socket *before any packet exists*. Not a delayed packet — no connection state at all. The program is small enough that the instruction budget is a design constraint, not an afterthought.

**Attribution.** A node-level interceptor binds the held socket to five identifiers: kernel socket cookie, process, executable digest, cgroup, and pod identity. It then parks the first bytes the agent writes, so there is content to judge before the handshake completes.

**The verdict.** A verifier in a *separate container with its own key* receives the request and returns a signed verdict that names exactly one socket cookie and one destination. The verdict must be unexpired and unused. No verdict by the deadline → connection reset. Late approval → refused. The verifier's key is never held by the enforcing side, so compromising the enforcer does not let you mint approvals.

**The recursion fix.** The agent's attempt to reach the verifier is itself escrowed — otherwise the first thing a compromised agent does is talk to the thing that approves its traffic.

**Content gating without TLS termination.** An eBPF uprobe at the agent's TLS-library boundary reads plaintext before encryption: shared OpenSSL, static OpenSSL, and Go `crypto/tls` are all covered. A model scores the request body, the connection is held on that verdict, TLS stays end-to-end, and the enforcer holds no key.

**Threat model coverage.** A 36-check adversarial suite run against real cgroups: forged verdicts, rebound verdicts (verdict for socket A replayed against socket B), replayed verdicts, expired verdicts, late verdicts, child-process bypass, renamed-binary bypass, and UDP. The same 33 applicable checks also pass against an iptables-based hold point, which is a useful signal that the guarantee comes from the *placement* rather than from eBPF specifically.

## Key Results

| Metric | Result |
|---|---|
| Adversarial checks passed | 36/36 on the eBPF hold point (33/33 applicable on the iptables hold point) |
| Added latency per released connection | 4.5 ms median |
| Of which policy evaluation | 24 µs |
| Kubernetes validation | 600 connections from 6 pods gated; unmodified pods attributed to SPIFFE identity |
| Raw socket bypassing a sidecar proxy | held |
| UDP | pinned to the resolver |

The latency number is the one that decides adoption: 4.5 ms median on a connection that is about to do a TLS handshake (tens of ms) and an LLM call (hundreds to thousands of ms) is noise. 24 µs of that being the actual policy decision means the overhead is dominated by the hold-and-release machinery, not the judgment.

## What's Novel

1. **Approval bound to a socket cookie, not to a request.** This is the whole idea. A verdict is single-use, expiring, destination-pinned, and cryptographically signed — so "approved" becomes a capability rather than a log line.
2. **Enforcement below the process.** Because the hold is in `connect()`, child processes, renamed binaries and raw sockets inherit it. That closes the entire class of bypasses that application-layer controls cannot.
3. **Verifier isolation with key separation.** The enforcement point cannot mint verdicts. This is a proper two-party design, not a policy engine with a bypass flag.
4. **Pre-encryption content inspection without MITM.** The TLS uprobe gives you plaintext at the library boundary while preserving end-to-end TLS and holding no keys — sidestepping both the "break TLS" and "you can't see anything" failure modes.
5. **Deadline-defaults-to-deny.** No verdict by deadline resets the connection. Fail-closed at the primitive level rather than in configuration.

## My Connection

This is the enforcement answer to a question that keeps coming up in agent security: *where do you put the control so the agent cannot route around it?* Everything MCP-side — tool allowlists, response scanning, OAuth scoping — lives at the tool boundary, which the agent's own process can bypass if it has a shell or a socket. Connection Escrow says: stop trying to control the agent, control the syscall.

The `SPIFFE identity` attribution is directly reusable for MCP gateways running in Kubernetes: you get per-pod identity for free and can bind tool-level authorization to workload identity rather than to a bearer token the agent holds. And the "verdict names one socket cookie" property is exactly what a tool-call approval needs — an approval for `read_file(/tmp/x)` should not be replayable as an approval for `read_file(~/.ssh/id_rsa)`.

## What I Learned

Agent egress control fails not because the policy is wrong but because the *placement* is wrong. If the control sits above the socket, a determined or merely buggy agent opens its own socket and you never see it. Move the control into `connect()` and the bypasses disappear by construction — a child process cannot escape a cgroup hook.

The second lesson is about what "approval" should mean. In most agent systems, approval is a message in a conversation: the human says yes, and the agent does whatever it wants next. Here approval is a signed, expiring, single-use token naming one destination — which is the difference between "the user consented" and "the system cannot do otherwise." That distinction is the whole ballgame when the thing you are governing is non-deterministic.
