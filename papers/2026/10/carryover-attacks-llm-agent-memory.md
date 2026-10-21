# After the Attacker Leaves: Benchmarking Carryover Attacks on LLM Agents

**Authors:** Zhiyue Xu, Xukai Wang, Zhiyuan Huang, Mingjie Zhan, Jian Zhao, Xu-Yao Zhang
**Institution:** Not listed on the arXiv abstract page
**Published:** October 20, 2026 — accepted at **EMNLP 2026 Main Conference**
**Link:** https://arxiv.org/abs/2610.28425
**Code:** Not linked in the abstract

---

## Problem Statement

Modern LLM agents persist state across sessions through two channels: **long-lived memory files** that get prepended to the system prompt, and **scheduled tasks** that wake the agent without any user present. A one-time indirect prompt injection that lands in either channel does not die with the session. It re-fires — later, repeatedly, and with no attacker present.

The authors call these **carryover attacks**. The gap they identify is methodological: every existing prompt-injection benchmark scores an attack in the single session where the malicious content first appears. Nobody measures the gate that governs *when and how often* a planted payload re-fires afterwards.

## Methodology

They build **CarryoverBench**, structured to separate two stages that prior benchmarks conflate:

- **Planting** — the injection gets committed to memory or to a scheduled task.
- **Triggering** — the payload actually influences behavior in a later session.

**Scope:** 12 personal-assistant scenarios, 101 attack tasks, across three trigger channels:

| Channel | Mechanism |
|---|---|
| **Instant** | Same-turn indirect prompt injection (the standard case) |
| **Autonomous** | Heartbeat-scheduled task wakes the agent with no user input |
| **Latent** | Memory resurfaces the payload in a later session |

A **rule-based verifier** scores planting and triggering as independent stages, which is what lets the paper say whether a defense stopped the attack or merely stopped the symptom. Evaluation covers **five frontier models**, and the Latent channel is ported to a second runtime — **Hermes** — to test whether the effect is an artifact of one harness.

## Key Results

- **No decay.** Across five rounds, planted payloads re-fire without measurable degradation. The second, third and fifth re-firings are roughly as effective as the first.
- **The trigger stage is not a meaningful filter.** Conditional on successful planting, moving from planting to triggering "often provides little additional protection" — the hard part is committing the payload, not setting it off.
- **Runtime portability.** Porting the Latent channel to Hermes *attenuates but does not eliminate* the effect. The authors read this as evidence that carryover defense needs **runtime-layer controls**, not just prompt-level interventions.
- The Instant channel is included as a baseline so the carryover channels can be compared against the case everyone already measures.

## What's Novel

1. **Naming the persistence problem as an attack class.** Prompt injection has been treated as a per-session event; this reframes it as a *durable* compromise of agent state, closer to a persistence mechanism in classic intrusion models than to a bad prompt.
2. **Two-stage scoring.** Separating planting from triggering is what makes the "no decay" finding legible — and it shows that defenses which only harden the trigger are defending the easy half.
3. **Cross-runtime replication, including Hermes.** Most agent-security papers evaluate one harness and generalize. This one explicitly checks whether the effect survives a runtime change, and reports that it partly does.

## My Connection

This is directly load-bearing for Manny's setup. He runs Hermes with persistent memory plus a daily cron fleet — meaning both channels in this paper are live in production: memory files that get prepended to the system prompt, and scheduled jobs that execute with no human in the loop.

Three concrete implications:

- **Cron prompts are an injection target.** Anything a scheduled job reads (web pages, issue trackers, email, tool output) can write into the next run's context. The digest pipeline itself reads untrusted pages.
- **A clean session is not a clean agent.** Incident response has to include "what is in memory and what is in the task queue", not just "what happened this session."
- **Runtime beats prompt.** The paper's own conclusion — runtime-layer controls — points at the same design as ClawMMU and connection-escrow work: enforce at the boundary where content enters context, not in the instruction text.

## What I Learned

The dangerous half of a prompt injection is the *write*, not the *read*. Once a payload is committed to memory or a scheduled task, it becomes a standing instruction with the agent's own authority, and re-firing costs the attacker nothing. The practical rule: treat every write into persistent agent state as a privileged operation requiring provenance, and treat "it worked fine in this session" as no evidence at all.
