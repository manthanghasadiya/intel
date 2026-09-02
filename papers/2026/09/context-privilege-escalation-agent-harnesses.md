# Context Privilege Escalation Attacks against AI Agent Harness

- **Paper:** What's in Your Agent's Context? Context Privilege Escalation Attacks against AI Agent Harness
- **Link:** https://arxiv.org/abs/2609.01222
- **Authors:** Zichuan Li, Jian Cui, Ashley Chen, Xiaojing Liao, Luyi Xing
- **Institution:** Indiana University Bloomington (Liao & Xing security groups)
- **Published:** September 1, 2026 (arXiv:2609.01222)
- **Code:** Not linked on the abstract page at digest time

## Problem Statement (plain English)

An AI coding agent's "context" is assembled from many places: the system prompt, conversation turns, files it reads, tool outputs (MCP included), terminal logs, transcripts. Those sources have very different trust levels — but the harness stuffs them into one flat window where the model treats them with similar authority. This paper asks: can an attacker smuggle content into a *low-privilege* context slot and have it *escalate* to a higher-privilege position — like a user-role message gaining the authority of a system/developer message, or a throwaway file read becoming a persistent instruction? That is a privilege escalation problem, and until now nobody had studied it systematically across real harnesses.

## Methodology (technical)

- Audited **12 AI agent harnesses** (including Claude Code and OpenAI Codex) by analyzing how each one assembles context: which sources exist, what role/priority each source gets, and how content can move between sources.
- Defined two new attack classes:
  - **Message-role context privilege escalation (M-CPE):** adversarial content enters at a low-privilege message role and escalates to a higher-authority role (e.g., user content gaining system-level steering power over the model's behavior).
  - **Cross-source context privilege escalation (X-CPE):** adversarial content is propagated from its entry source into a *more persistent* or *broader-scope* context source (e.g., from a one-off tool result into a memory/summary/transcript that influences all future turns).
- Mapped **282 vulnerable context sources** across the 12 harnesses and constructed end-to-end attack chains from a single injection to full agent compromise.

## Key Results (with numbers)

- 12 harnesses audited; **282 vulnerable context sources** identified.
- Both M-CPE and X-CPE demonstrated against mainstream harnesses including Claude Code and Codex.
- Attack chains reach **full agent compromise** (persistent steering, credential/tool access, long-lived malicious behavior) rather than single-turn injection.

## What's Novel

Prior prompt-injection work treats the model as the vulnerable component. This paper moves the target to the **harness**: how context is gathered, role-tagged, persisted, and re-injected is itself the privilege boundary. Naming two concrete escalation classes (M-CPE, X-CPE) gives researchers and defenders a vocabulary and an audit method — plus a measured inventory (282 sources) of where the boundaries are weak in today's most-used tools.

## My Connection (to Manny's work)

Manny operates agent harnesses daily — Claude Code, MCP servers, custom tool chains. This paper is effectively a pre-built red-team checklist: every file read, MCP tool result, environment dump, and log transcript is a candidate CPE primitive. It also sharpens the threat model for the recent Apify MCP prompt-injection work and the METR API-key theft (a vibe-coded dashboard's context let an attacker extract a key): both are X-CPE-flavored failures in the wild. Defensively, it argues for per-source trust tagging and role separation inside the context window — controls most harnesses still don't expose.

## What I Learned (plain English)

An agent's context window is not a flat text box — it's a stack of sources with different privileges, and the harness decides the boundaries. If a harness lets content flow from a low-trust slot (a webpage, a tool result) into a high-trust, persistent slot (memory, summary, system-adjacent transcript), then one injection becomes permanent control. Auditing an agent for security means auditing its *context-assembly pipeline*, not just the model's safety filters.
