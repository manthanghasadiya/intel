# SoK: When Safe Agents Fail Together — Multi-Agent LLM System Security

- **Paper:** SoK: When Safe Agents Fail Together: The Security of Multi-Agent LLM Systems
- **Link:** https://arxiv.org/abs/2609.00595
- **Authors:** Rui Yang, Junjie Xu, Zhengyu Liu, Neil Fendley, Yang Hong, Ziyang Li, Yinzhi Cao
- **Institution:** Yinzhi Cao's group, Johns Hopkins University, with collaborators
- **Published:** September 1, 2026 (arXiv:2609.00595, cs.CR / cs.AI, 21 pages)
- **Code:** Not linked on the abstract page at digest time

## Problem Statement (plain English)

Teams are wiring LLM agents into multi-agent systems (MAS): one agent plans, another browses, another writes code, another approves actions. Each individual agent may be perfectly safe — good system prompt, guardrails, tool allow-lists. The paper's core claim is in the title: **safe agents can fail together**. When agents exchange messages, share state, delegate authority, and hand off tool access across organizational boundaries, failures emerge that no single agent's local checks can catch. And because those failures are invisible unless you watch the actual execution flow, researchers and vendors keep misattributing them.

## Methodology (technical)

- A **systematization of knowledge (SoK)** — a structured survey — of multi-agent LLM security, organized around an **execution-centered analysis** rather than per-agent properties.
- Examines how MAS move four things across principal boundaries: **information, state, decisions, and authority**.
- Analyzes existing multi-agent attack/defense literature through this execution-level lens, taxonomizing where local (per-agent) safety checks break down and what systemic failures look like.
- Argues methodological point: without an execution-level view, a failure observed in a multi-agent setting can easily be mistaken for evidence of a genuinely multi-agent security effect (i.e., flawed experimental attribution).

## Key Results (with numbers)

- A systematized taxonomy of multi-agent security failures organized by what crosses boundaries (information / state / decisions / authority).
- An execution-centered framework for evaluating MAS security claims.
- Documentation of the attribution pitfall: single-agent-looking failures that are actually emergent from agent-to-agent interaction (and vice versa), with guidance for sound evaluation design.

## What's Novel

Most agent-security work is single-agent: prompt injection against one chatbot, one coding agent, one MCP tool chain. This SoK moves the unit of analysis to the *system* — and makes a sharp methodological contribution: it tells researchers how to *tell* whether a failure is genuinely multi-agent or just a local failure observed in a multi-agent setup. That matters because vendors keep shipping "multi-agent security" features based on misattributed evidence.

## My Connection (to Manny's work)

Manny's world is increasingly multi-agent: Claude Code spawning subagents, MCP servers mediating tools between agents, orchestrators delegating to specialist agents. This SoK is the map of where authority crosses trust boundaries in such stacks — directly relevant to threat-modeling the agent-vs-agent attack surface (one compromised agent steering another, shared context poisoning, delegated-tool abuse). It pairs with the Context-PE paper: CPE is a harness-level mechanism; this SoK is the system-level theory of why local checks fail.

## What I Learned (plain English)

"Each agent is safe" does not imply "the system of agents is safe." The dangerous moments are the handoffs — when information, state, decisions, or authority cross from one agent's control to another's. If you want to secure an agentic system, you have to watch the execution flow between agents, not just audit each agent in isolation. And when you see an agent doing something bad, check whether the cause was actually another agent's influence before you blame (or fix) the wrong component.
