# When Agents Look Like Beacons: NIDS Evasion by Model Context Protocol Traffic

**Author:** Muhammad Abdullah Sohail
**Published:** 2026-09-16 (arXiv v1)
**Link:** https://arxiv.org/abs/2609.19091
**Code:** Not stated in abstract (proposes a standard rather than shipping an artifact)

---

## Problem Statement (plain English)

Model Context Protocol (MCP) is how AI agents talk to remote tools, and it does so over Streamable HTTP using JSON-RPC — machine-generated, authenticated, high-frequency, tightly-polled traffic. Enterprise defenders have always used "machines talking on a regular schedule" as a red flag for malware command-and-control (C2) — that's the whole premise of beacon detection (Cobalt Strike-style polling). So the obvious question: when agents start living inside the enterprise network, do our intrusion detection systems see their MCP chatter as C2 — or do they see nothing at all?

The paper's finding is the uncomfortable one: they see **nothing at all**. MCP traffic is structurally and temporally identical to C2 beaconing, but because detection heuristics are tuned for *traditional malware* (not for LLM reasoning loops), the traffic slides straight through — a visibility gap, not a false-positive problem.

## Methodology (technical)

- Controlled **Docker-based testbed** generating **11 mathematically defined traffic profiles**.
- Each profile evaluated under **three TLS conditions**: Opaque TLS, TLS-Inspected, and Cleartext.
- Detection stack under test: **Suricata** (signature matching) and **RITA** (behavioral beacon scoring), using the **Emerging Threats (ET) Open** ruleset.
- Independent variables included **temporal smearing (jitter)** applied to inter-arrival times.
- The paper notes MCP traffic exhibits **lognormal inter-arrival distributions** characteristic of generative-AI reasoning loops — deliberately distinct from the periodic/constant-cadence distributions beacon-scoring heuristics are tuned to catch.

## Key Results (with numbers)

- **0.0 behavioral beacon score** across all tested MCP JSON-RPC profiles — RITA never flagged the traffic, regardless of jitter or TLS condition.
- **Near-zero Suricata content alerts** under ET Open rules.
- **TLS inspection did not help.** Opaque TLS does expose the flow to *flow-level temporal analysis* (removing content visibility but leaving cadence visible) — yet the heuristics still failed, because they look for traditional malware cadence, not lognormal reasoning-loop cadence.
- **Jitter did not help detection** either — the evasion held whether traffic was smeared or not.
- Structural finding: MCP's authenticated, high-frequency JSON-RPC poll pattern is a *false negative* for C2 detectors by construction.

## What's Novel

1. **Reframes MCP as a covert-channel-shaped problem rather than a prompt-injection problem.** Most MCP security work is about tool poisoning / injection at the protocol layer; this is about the *network layer* and defenders' blindness to it.
2. **Demonstrates the detection failure empirically across TLS postures and jitter levels** — closing the obvious objection ("you're just looking at encrypted traffic") by testing cleartext too.
3. **Proposes an agent-native network indication standard**: an Agent-Native ALPN and standardized out-of-band headers, so agent traffic can be positively identified and *separated* from C2, rather than relying on heuristics that can't distinguish them.
4. **Identifies the mechanism of failure precisely** — the mismatch between lognormal inter-arrival distributions (AI reasoning) and malware-cadence heuristics — rather than vaguely reporting "IDS doesn't catch it."

## My Connection (to Manny's work)

This is the network-layer twin of the agent-visibility problem Manny already covers at the tool layer. If NIDS is blind to MCP, then:
- **Red-team content angle:** "Your IDS can't see MCP — here's why, and here's the 11-profile testbed that proves it." Highly demonstrable, and the 0.0 score is a screenshot-ready headline number.
- **Defense angle:** it justifies the "log the agent network, not just the agent" argument and gives a concrete standards ask (Agent-Native ALPN) that Manny can cite when talking about agent observability.
- Pairs directly with the same author's companion paper on remote-MCP centralization (arXiv 2609.19100) — together they form a two-part argument that the MCP ecosystem is both **concentrated** and **invisible**.

## What I Learned (plain English)

The scariest part of agent infrastructure isn't that attackers can hijack it — it's that when it runs normally, it already looks exactly like the thing we're supposed to be hunting, and our detection stack has decided not to care. If MCP is indistinguishable from C2 and NIDS flags neither, then "we monitor the network" is not a control anymore. You can't detect the anomaly when the anomaly is the baseline. The fix isn't better signatures — it's a way for agents to *announce themselves* in a form machines can verify.
