# SynChain: Inducing Computer-Use Agent Systems to Construct Their Own Attack Chains

- **Authors:** Fuyao Zhang, Jiaming Zhang, Che Wang, Boyang Chen, Yurong Hao, Xiongtao Sun, Guowei Guan, Blaise Delattre, Yang Cao, Wei Yang Bryan Lim
- **Institution:** Not listed on arXiv abstract page
- **Published:** Submitted Aug 7, 2026; announced Aug 10, 2026
- **Link:** https://arxiv.org/abs/2608.06862
- **Code:** Not stated in abstract

## Problem Statement (plain English)

Computer-use agents don't just act — they *create* things: skills, memory entries, scripts, configs — and reuse them later. Security research has mostly assumed attacks are externally triggered (a malicious page, a poisoned prompt) or bounded in time. This paper attacks the gap in between: what if the compromise lives *inside the artifacts the agent itself synthesizes*? Malicious influence is hidden in the structural redundancies of those artifacts, survives the agent's own state updates and vetting, and reactivates days later as "trusted context" — with no new malicious input at execution time. The agent becomes its own attack chain.

## Methodology (technical)

- **SynChain paradigm:** a self-synthesized attack using **persistence-aware directed supervised fine-tuning (SFT)** to induce agents to create poisoned yet benign-looking artifacts during normal task execution.
- **CUAChain dataset:** 30 benign task chains plus three attack objectives, used to train and evaluate the induction.
- **Attack flow:** dormant payloads embedded in synthesized artifacts reactivate in future workflows as trusted context, operating entirely without new malicious exogenous inputs.
- **Evaluation:** OpenClaw, Codex, and Claude Code under **four defense settings**, compared against adapted baselines.

## Key Results (with numbers)

- SynChain achieves **high attack success** on OpenClaw, Codex, and Claude Code across all four defense settings.
- **Outperforms adapted baselines** — the paper doesn't disclose exact percentages, but the claim is consistent superiority over existing attack/poisoning baselines.
- Demonstrates the attack works with **zero new malicious inputs** at trigger time: the payload rides entirely on previously synthesized, vetted artifacts.

## What's Novel

- First demonstration of **self-propagating compromise through the agent's own synthesized artifacts** — no external trigger, no temporally-bounded attack window.
- Formalizes the threat as a training-time induction problem (persistence-aware directed SFT), not just a runtime injection.
- Shows standard vetting of skills/memory is insufficient because the poison lives in *structural redundancies* that survive updates — a new class of "artifact laundering."

## My Connection (to Manny's work)

This is the modern supply-chain attack for agent ecosystems: skills and MCP tool definitions are the new npm packages, and they're often agent-written. For Manny's red-team work it provides a concrete playbook (induce → poison artifact → reactivate later) to test against self-evolving agents and skill marketplaces; for defensive work it makes the case for **provenance-aware reasoning over cross-task execution trajectories** — the same conclusion as the earlier SkillJack/SkillTrace line, now with an induction-based attack vector.

## What I Learned (plain English)

An agent's persistent state is a double-edged sword: the artifacts it writes today are the attack vector of tomorrow. If you can't trace where every skill, memory entry, and tool definition came from — and whether it was modified after creation — you can't trust anything the agent reuses later. Provenance tracking across task boundaries is becoming the single most important defensive primitive for agent systems.
