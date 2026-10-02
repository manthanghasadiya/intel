# The Innocent Courier: Covert Exfiltration Through Legitimate LLM Web Fetching

**Authors:** Alessandro Pegoraro, Daryan Merx, Phillip Rieger, Ahmad-Reza Sadeghi
**Institution:** Technical University of Darmstadt (System Security Lab)
**Published:** Oct 1, 2026
**Link:** https://arxiv.org/abs/2610.01768
**Venue:** arXiv:2610.01768 [cs.CR]

---

## Problem Statement (plain English)

Local malware usually can't talk to the internet directly — network libraries are restricted and obvious "send this data out" code is easy to catch. But the user's LLM assistant *can* reach the internet (it fetches web pages, docs, packages). LLMLeak shows that a piece of malicious local software can **borrow the LLM's own web-fetch tool as a courier**, hiding stolen data in a URL that the model deliberately requests, without ever opening a socket itself. Existing defenses (prompt-injection structuring, running local LLMs to avoid sharing data with the provider) don't address *this* direction: leaking confidential data **to a third party**.

## Methodology (technical)

- **Threat model:** a malicious component on the client side that cannot communicate directly with the internet.
- **Mechanism:** the malware embeds a **secret into a URL** and presents that URL to the LLM as a site that "provides information required for a benign task" (e.g. migrating a software library). The LLM, doing its job, calls its web-fetch tool on the URL. When the request reaches an **attacker-controlled DNS or web server**, the attacker reads the encoded secret out of the request path/hostname.
- The attack deliberately avoids asking the model to *generate network code* (a well-known, easily-detected pattern) and relies only on the ordinary fetch tool — so it's "the innocent courier" carrying the payload.
- **Evaluation:** eleven open-parameter models, plus a case study on real-world chatbot deployments.

## Key Results (with numbers)

- **Attack success rate: 79.7%** across eleven open-parameter models.
- Real-world chatbot case study demonstrates the channel is practical, not merely theoretical.
- The channel uses **only the model's legitimate fetch capability** — no direct network access, no code generation, no exfiltration API.

## What's Novel

1. Reframes the web-fetch tool as an **exfiltration primitive**, not just a prompt-injection vector (injection is about getting *in*; LLMLeak is about getting *out* and also *in* — a bidirectional covert channel).
2. Specifically bypasses the common mitigations (structured inputs; local-only LLMs) because the leak target is a third-party server, not the model provider.
3. Shows the attack survives when the malware "cannot communicate directly with the internet," which is exactly the isolation local-LLM deployments advertise.

## My Connection (to Manny's work)

- This is a direct answer to "I run a local model so my data can't leak" — the fetch tool still leaks it. Great counterpoint for agent-security advisories and for MCP tool design (any `fetch`/`browse`/`get_url` tool is a potential courier).
- Defensive takeaway to socialize: **egress allow-listing and URL/DNS logging for agent tool calls**, not just prompt-injection filters.
- Material for a demo/content piece: a benign-looking "read this migration doc" fetch carrying an encoded secret.

## What I Learned (plain English)

A tool that's designed to bring information *in* can be turned around to carry information *out*. If your assistant can open a URL you didn't type, an attacker who controls what it "needs" to look up can make it call back to themselves — and the data rides out in the request itself. Blocking direct network access for local processes isn't enough; you have to watch what URLs the model is coaxed into fetching.
