# Recuris: Recursive Experiential-Working Memory Evolution for Long-Horizon Agent Harnesses

- **Paper:** [arXiv:2608.24876](https://arxiv.org/abs/2608.24876) · [HF paper page](https://huggingface.co/papers/2608.24876)
- **Authors:** Zhaochen Yu, Yingcheng Wu, Zhenfei Yin, Kaiyuan Chen, Zhe Zhao, Mengdi Wang, Shuicheng Yan, Ling Yang
- **Published:** Aug 25, 2026
- **Code:** available (per arXiv listing)
- **Subjects:** cs.AI, cs.CL

## Problem Statement (plain English)

Long-horizon agent tasks (multi-hour, multi-step work) are where recursive self-improvement (RSI) falls apart. As an agent works, its history grows, and that growing blob of context obscures what the task actually needs right now — so the agent keeps invoking the wrong skills at the wrong time. Memory, in other words, is the bottleneck: everything is remembered, nothing is *situated*. Recuris is an architecture that separates "what I know" from "what I'm doing now" and lets the second one drive the first.

## Methodology (technical)

- Two-tier memory: **Experiential Memory** (long-term store of skills, procedures, and knowledge) and **Working Memory** (live representation of current task state and progress).
- Recursive loop: Working Memory tracks task progress and *guides skill selection* from Experiential Memory, grounding each skill invocation in current needs — so skill use is chosen against the live task state instead of stale history.
- The recursion: outcomes of executed skills feed back into both memory tiers, evolving them over the horizon (hence "memory evolution").
- Evaluated in long-horizon agent-harness settings where standard single-context agents degrade (per the abstract's framing of growing histories obscuring state and misaligning skill invocation).

## Key Results

- Recuris is designed to keep recursive self-improvement tractable as task histories grow — the specific failure mode addressed is skill invocation misalignment over long horizons.
- Code is public for reproduction.
- Exact benchmark numbers weren't in the abstract/announcement; the contribution is the architecture and the memory-split thesis.

## What's Novel

Most agent memory work treats memory as a retrieval problem (what to fetch). Recuris treats it as a *control* problem: working memory doesn't just store state, it actively gates which long-term skills are even eligible for invocation. That "memory-as-controller" framing — plus the recursive evolution loop — is the differentiator versus RAG-style or single-context baselines.

## My Connection (Manny's work)

Memory is the new attack surface for agents. Yesterday's digest covered InjecMEM (memory injection attacks on LLM agent memory systems); Recuris is the architectural counter-side — a structured memory design where poisoning surface is split across two tiers. If working memory gates skill selection, an attacker who poisons experiential memory (the long-term store) may get indirect control over which skills fire — a subtler version of prompt injection. Worth testing: does the working/experiential split actually *reduce* exploitability, or just move the target?

## What I Learned (plain English)

1. "Long context" is not "good memory" — unstructured history grows into noise; agents need memory that is *situated* in the current task.
2. Separating "what the agent knows" from "what the agent is doing" is a design decision with security consequences: different tiers = different poisoning targets.
3. Recursive self-improvement fails on memory management before it fails on capability — a useful lens for evaluating agent harnesses.
