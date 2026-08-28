# RedEvoAgent: Automatic Red-Teaming Agent with Experience-Driven Skill Evolution

- **Link:** https://arxiv.org/abs/2608.27439
- **Authors:** Junjie Zhang, Hui Liu, Kecheng Chen, Xianbo Mo, Changsheng Chen, Haoliang Li
- **Institution:** Not listed in arXiv metadata (senior author Haoliang Li is at City University of Hong Kong)
- **Published:** 2026-08-27
- **Code:** Not listed

## Problem Statement (plain English)

LLM agents now run inside real product harnesses where a jailbreak means harmful *tool use* and persistent state changes — not just bad text. Prior automatic red-teaming either uses fixed attacks (stale, easily blocked) or replays full attack trajectories from previous successful jailbreaks. Trajectory replay has two flaws: it can retrieve misleading experiences (retrieval bias, unclear which tool actually caused the win), and full trajectories burn context and are hard for a human to interpret. We want an attacker that gets better over time and explains itself.

## Methodology (technical)

RedEvoAgent is a black-box red-teaming agent with three components:

1. **Skill distillation** — compresses cross-case attack trajectories into a single concise, human-readable "attack skill" (a reusable recipe), instead of dumping raw trajectories into context.
2. **Tool-effectiveness profiling + Deciding-Tool Attribution** — after each run, attributes the outcome to the specific tool(s) that mattered, and updates the skill accordingly so credit isn't given to irrelevant steps.
3. **Validation ratchet** — a skill update is only retained if it improves performance on a validation set; regressions are rolled back.

The skill is the persistent attacker memory; it evolves across cases and is carried between attacks.

## Key Results (with numbers)

- Outperforms fixed-attack and trajectory-retrieval agentic baselines across multiple benchmarks, target models, and target execution harnesses.
- Improves tool efficiency (fewer redundant calls than trajectory replay).
- Transfers across attacker models and across target execution harnesses — a skill learned attacking model A on harness X still works attacking model B on harness Y.

## What's Novel

- Moving from *trajectories* to *distilled skills* as the unit of red-team experience — fixes retrieval bias and unclear tool credit, cuts context overhead, and produces something a human operator can read and audit.
- The Deciding-Tool Attribution + validation-ratchet loop gives principled credit assignment and safe self-improvement, addressing a known failure mode of agentic attackers.

## My Connection (to Manny's work)

This is a directly usable offensive primitive: instead of hand-writing jailbreak chains, Manny can run RedEvoAgent-style skill evolution against target harnesses (Claude Code, Kiro, n8n, MCP servers) to auto-generate a maintained library of human-readable attack skills. The interpretable-skill design also means findings translate into defensive guidance (skill → detection rule) without re-deriving from raw logs.

## What I Learned (plain English)

- The best "memory" for an automated attacker isn't raw transcripts — it's a compact, evolving, human-readable recipe of what worked and which tool earned the win.
- Self-improving attackers need a validation ratchet: only keep updates that provably improve outcomes, or the skill set degrades into superstition.
- Attack skills learned against one model/harness pair transfer to others — so a good red-team skill library is a durable asset, not a one-off artifact.
