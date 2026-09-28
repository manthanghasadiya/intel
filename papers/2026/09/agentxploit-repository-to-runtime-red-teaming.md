# AgentXploit: Autonomous Repository-to-Runtime Red-Teaming for AI Agents

**Authors:** Weida Liang, Shi Qiu, Zhun Wang, Simon Sure, Xiaoyuan Liu, Tianneng Shi, Zhaorun Chen, Wenbo Guo, Dawn Song
**Institution:** UC Berkeley and collaborators (Dawn Song's group)
**Date:** Submitted Sep 25, 2026 · cs.CR, cs.AI
**Link:** https://arxiv.org/abs/2609.31318
**Code/Benchmark:** AgentXploit-Bench (72 reproducible vulnerabilities across 12 open-source AI-agent systems/frameworks)

---

## Problem Statement (plain English)

AI agents are just language models wired to tools that can edit files, call APIs and run code. That creates two very different ways to break one:

1. **Adversarial content** changes what the agent decides to do (prompt injection, poisoned context).
2. **The surrounding software is buggy** — path traversal, command injection — and the agent becomes the delivery vehicle.

Auditing this before deployment is awkward. Existing tooling tends to be either *static* (read the repo, guess at code paths) or *dynamic* (poke a running system and hope). Neither reliably closes the loop from "this input reaches this dangerous sink" to "here is a working, verified exploit."

The paper carves out a precise, legitimate scope: **authorized white-box pre-deployment auditing** — the auditor has the repository and a controlled runtime, but any successful attack must still go through the task's defined attacker interface and be **confirmed by an external verifier**. No hand-waving, no "the model said it might work."

## Methodology (technical)

AgentXploit splits the job across two specialised roles, because discovery and exploitation turn out to be genuinely different problems:

- **Analyzer Agent (repository-level):** traces attacker-controlled inputs through the codebase to sensitive operations and records **code-supported candidate attack paths**. Its output is grounded in the source, not in model intuition.
- **Exploiter Agent (runtime-level):** takes those candidate paths and turns them into concrete attacks, then **revises them using runtime feedback** — a closed loop, not a one-shot generation.

To measure this, the authors built **AgentXploit-Bench**: 72 reproducible vulnerabilities spanning 12 open-source AI-agent systems and frameworks. The design deliberately requires external verification, so a "successful" attack has to actually demonstrate the effect rather than produce plausible text.

They compare against a strong general coding-agent baseline (**Codex**) and against a dedicated injection attacker (**AgentVigil**) on AgentDojo.

## Key Results (with numbers)

- **End-to-end success: 59.3%** across three runs, versus **38.4% for Codex**.
- Under a **token-budget-matched** comparison (so Codex isn't simply starved), Codex reaches **46.3%** — AgentXploit still ahead.
- On **AgentDojo**, where injection points are supplied rather than discovered, the **Exploiter Agent hits 79.2% attack success** versus **52.7% for AgentVigil**.
- The headline interpretive finding: **repository discovery and runtime exploitation are distinct challenges.** Folding them into one agent underperforms separating them — the bottleneck moves depending on which half you're in.

## What's Novel

- **Role separation as the core design choice.** Prior work largely treats agent red-teaming as one model doing everything. AgentXploit shows the split is load-bearing, and quantifies the gap.
- **Externally verified attacks only.** Success is defined by a verifier, not by the attacking model's own claim — which is exactly the failure mode (models misreporting their own actions) that has been showing up in frontier-lab incident reports.
- **A reproducible benchmark (AgentXploit-Bench)** with 72 concrete, verifiable vulnerabilities in real agent frameworks — rare in a field where most "agent vuln" papers use synthetic toys.
- **Honest scope:** the paper is about *authorized* white-box auditing, and says so. It's an auditing capability, not a weaponized exploit generator aimed at third parties.

## My Connection (to Manny's work)

This is the most directly usable agent-security paper in weeks for red-team engagements:

- **It is literally a pentest methodology.** Analyzer-then-Exploiter is how a human tester already works: map tainted input to sinks, then prove impact. AgentXploit automates the boring 80% and keeps the verifier honest.
- **The benchmark is a ready-made evaluation harness.** If you're assessing a client's agent codebase, AgentXploit-Bench gives you a scored, comparable result rather than an anecdote — and the 12-framework spread means most stacks are represented.
- **Pairs with today's defense-side papers.** AGATE (2609.30830) and the NVIDIA OpenShell work both assume the agent *will* be driven off-task by adversarial content; AgentXploit is the tool that demonstrates it under controlled, authorized conditions. Offense scores the risk, defense bounds it.
- **Auditing hygiene, not just exploitation.** Because attacks must traverse the task-defined attacker interface and pass an external verifier, the output is defensible in a report — you can show the reproduction, not just a model transcript.

## What I Learned (plain English)

Splitting a hard security job into "find the path" and "prove the path" makes both halves better — the Analyzer stops hallucinating exploits and the Exploiter stops wasting budget on dead ends. The number that matters most isn't the 59.3% headline; it's that a general-purpose coding agent (Codex) gets 38–46% on the same task. That is the uncomfortable part: **generic coding agents, pointed at an agent codebase, are already roughly half of an autonomous red-teamer.** If you build agents, the threat model is not "a determined human uses a vuln scanner" — it's "someone points a competent coding agent at your repo and it finds and fires real bugs about half the time." One sentence: *agent-on-agent auditing works, and you should be running it on yourself before someone else does.*
