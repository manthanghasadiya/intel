# SkillJack: Persistent Skill Backdoors in Self-Evolving Agents

- **Paper:** https://arxiv.org/abs/2608.03509
- **Authors:** Zonghao Ying, Xiangfan Wu, Huiyu Wu, Xing Zheng, Huangsheng Cheng, Xiaorong Shi, Jing Guo
- **Institution:** Tencent (code hosted under Tencent/AI-Infra-Guard)
- **Published:** Aug 4, 2026
- **Code:** https://github.com/Tencent/AI-Infra-Guard/research/skilljack

## Problem Statement (plain English)

Self-evolving agents (agents that turn past conversations into reusable "skills" they can call later) are being poisoned by attackers. Prior work poisoned the agent's *memory* or *retrieval* — but those attacks only work while the poisoned record is actively fetched. This paper finds a deeper problem: if you poison the agent's *experience*, the agent itself will distill that poison into a permanent skill — a durable behavioral artifact that no longer looks malicious and survives even after the original poisoned record is deleted. That's a backdoor with a self-cleaning trail.

## Methodology (technical)

SkillJack targets the experience-to-skill pipeline (e.g., SkillX, Anything2Skill):

1. **Implant** — inject poisoned trajectories into the agent's interaction history (via normal memory-poisoning vectors).
2. **Exploit the distillation** — when the agent summarizes experiences into skills, malicious intent gets laundered: the extraction process drops the harmful framing while keeping the harmful behavior (sanitization whitewashing).
3. **Persist** — the skill becomes a first-class artifact (cross-layer promotion), decoupled from its source records (persistence isolation).

The paper characterizes three properties: *sanitization whitewashing* (malice obscured during skill extraction), *cross-layer promotion* (transient experience → persistent capability), and *persistence isolation* (attack survives source-record deletion). Evaluated on SkillX and Anything2Skill with 150 trajectories across four policy-risk categories.

## Key Results (with numbers)

- **Detectability collapse:** safety detection drops from 98.5% (poisoned trajectories) to **11.4%** (extracted skills) on SkillX; Anything2Skill shows a similar effect.
- **Attack success:** 56.2% (SkillX) and 89.2% (Anything2Skill).
- **Persistence:** 80.0% of skill-mediated attacks survive deleting the original poisoned records.
- **Collateral:** some implanted skills unintentionally activate on benign queries.

## What's Novel

First attack on the *skill-formation* pipeline itself, rather than memory/retrieval. Key insight: the agent is the attacker's unwitting compiler — it transforms poison into clean-looking, persistent, self-cleaning artifacts. Prior defenses that scan memory or monitor retrieval miss the attack entirely because the payload no longer lives in either.

## My Connection (to Manny's work)

This is the supply-chain story for agent skills — the same class of artifact MCP servers and Claude Code skills represent. Anyone building agents that learn from history (or pulling third-party skills) inherits this attack surface. It pairs directly with the SkillSentry paper (dynamic skill scanning) and the same group's work; if Manny's tooling ingests or generates skills, provenance-aware skill lifecycle (signing, source tracking, re-scan after distillation) is the mitigation axis.

## What I Learned (plain English)

Distillation is not a sanitizer — it's a launderer. When an agent compresses experience into skills, safety filters that work on raw text stop working because the malicious payload has been rewritten by the agent itself. Deleting the original poison doesn't remove the backdoor; the skill is now an independent artifact. Any defense has to inspect the *output* of the learning process, not just the input.
