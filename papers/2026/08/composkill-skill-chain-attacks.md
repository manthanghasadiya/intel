# CompoSkill: Compositional Skill Chain Attacks from Individually Scanner-Passing LLM Agent Skills

- **Link:** https://arxiv.org/abs/2608.16246
- **Authors:** Mingxiao Liu, Zhoumian Jiang, Jianan Ma, Jian Zhang, Jialuo Chen, Xinhao Deng, Zhen Wang
- **Institution:** Not stated in listing
- **Published:** Aug 17, 2026
- **Code:** Not listed (9 pages, 5 figures)

## Problem Statement (plain English)

Agent skill marketplaces certify each skill *individually*: a scanner inspects a skill, returns a pass/fail verdict, and if every package passes, the ecosystem is declared safe. This paper shows that certification is path-level, not node-level: skills that are harmless on their own become dangerous when an agent chains them together — connecting one skill's outputs, capabilities, or side effects with another's. That's why existing per-skill scanners keep missing real attacks.

## Methodology (technical)

- **CompoSkill framework** with a dual attacker model:
  - **White-box attacker:** knows the victim's installed skill pool, injects explicit skill-id sequences to form a malicious chain.
  - **Black-box attacker:** knows only a role profile, downloads the top marketplace skills for that scenario, builds a **Skill Composition Graph**, and searches for high-risk chains whose implicit lures never name skill identifiers.
- **CompoSkill-Bench:** 1,140 records built from long-horizon professional workflows across 5 threat types and 6 scenarios, run on **OpenClaw** and **Nanobot** agent harnesses.
- Metrics: Chain Formation Rate (CFR), Attack Success Rate (ASR).

## Key Results (with numbers)

- **Chain Formation Rate:** up to **83.3%** (white-box), **80.6%** (black-box).
- Existing skill scanners block only a *limited fraction* of risky compositions.
- **Bridge-bonus-then-hop-decay:** a bridge skill increases attack success, but ASR degrades once the chain exceeds 3 skills — longer chains are less reliable.

## What's Novel

Reframes skill safety as a composition/path property rather than a per-package property — the first framework (plus benchmark) for constructing compositional skill chain attacks, with both white- and black-box attacker models and measurable CFR/ASR on real agent harnesses.

## My Connection (to Manny's work)

This is the direct continuation of the skill-scanner line (CollusKill, Aug 11 digest; MCP server skill injection): when Manny tests agent deployments, per-skill scanning reports are not a security boundary. The attack surface is the *graph* of skills an agent can reach — enumerate chains, not packages. Also: OpenClaw/Nanobot are the same harnesses used in production agent setups, so the numbers transfer.

## What I Learned (plain English)

- "Every package passes" is a meaningless guarantee — risk lives in how skills connect.
- A black-box attacker doesn't need to know the victim's skill IDs; role-based lure chains work almost as well (80.6% vs 83.3%).
- Chains longer than three skills self-destruct (hop decay) — defenders should focus on detecting 2–3 hop composition patterns, and attackers should keep chains short.
