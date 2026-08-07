# PoisonedEvolution: Trajectory Poisoning in Self-Evolving Agent Skill Systems

- **Paper:** https://arxiv.org/abs/2608.05563
- **Authors:** Jialuo Chen, Lingqi Jiang, Xinhao Deng, Xiaohu Du, Jianan Ma, Yunhao Feng, Yuqi Qing, Zhihao Yuan, Linkang Du, Jingyi Wang
- **Institution:** Zhejiang University (Wang's AI-security group)
- **Published:** 2026-08-06
- **Code:** None public at time of writing

---

## Problem Statement (plain English)

Self-evolving skill (SES) systems make agents permanently smarter: they watch agent trajectories (the step-by-step logs of how a task got done), distill the good ones into reusable "skills," and promote those skills into the agent's trusted skill bank. The whole point is that **untrusted experience becomes trusted instruction**. This paper asks: what if an attacker quietly contributes poisoned trajectories that look useful, get promoted into the skill bank, and then execute malicious behavior every time the skill runs? The attack surface isn't the model — it's the promotion pipeline that decides which experience becomes instruction.

## Methodology (technical)

- **Attacker model:** skill-visible black-box — can inspect a target skill and contribute bounded evidence (trajectory records), but cannot observe private trajectory pools or evolution logic, and cannot edit the skill bank directly.
- **Attack conditions:** the poison must satisfy three properties before promotion:
  1. **Inclusion** — the poisoned artifact must get into the candidate pool,
  2. **Evolution Attribution** — the target behavior must look *causally useful, recurrent, and generalizable* so the evolver credits it (the distinctive bottleneck),
  3. **Realization** — the behavior must actually fire when the promoted skill is used.
- **Evaluation:** four security-effect families tested with inert canary specifications (safe stand-ins that reveal when malicious behavior would trigger), against two structurally different SES pipelines: **SkillClaw** (six mainstream LLM evolvers) and **Trace2Skill**.

## Key Results (with numbers)

- **SkillClaw at 10% attacker support:** target behavior embedded in **546/600 trials (91.0% SER)** across six evolvers.
- **Trace2Skill at 10% support:** **369/600 (61.5% SER)** — attack transfers across evolution architectures, not just one pipeline.
- **Controlled study:** just **3 consistent attacker records in a 30-record batch** suffice for promotion; a single record is much weaker.
- **Ablations:** recurring support, causal framing, and domain-aligned encoding are the main determinants of success.

## What's Novel

Prior agent attacks target prompts, tools, or models. PoisonedEvolution is the first to attack the **trajectory → skill promotion process itself** — the curation layer that turns experience into trusted instruction. The "Evolution Attribution" bottleneck (making malicious behavior look causally useful) is a new, distinct attacker capability that no prior work modeled. Transfer across two different evolution architectures (91% → 61.5%) shows it's a class-level weakness, not a pipeline quirk.

## My Connection (to Manny's work)

Manny tracks agent supply-chain security — the SkillSpector scan, the malicious AI-skills trend, skill-bank poisoning. This paper supplies the missing half of that picture: even if a skill bank starts clean, its **promotion process** is a standing injection point that requires no code-level compromise. Any Manny client running self-evolving agents (auto-distilling trajectories into skills) inherits this exposure, and canary-spec testing is a cheap detection layer to add.

## What I Learned (plain English)

- The promotion/curation step in self-evolving agents is a security boundary, and it's the *weakest* one — an attacker with bounded, visible evidence can win 9 times out of 10 at only 10% support.
- Causal framing matters more than payload quality: poison works by looking useful, recurrent, and generalizable — which means defenders should scrutinize *consistent, recurring* contributions, not anomalous one-offs.
- "3 records in a 30-record batch" is the headline number to remember: promotion thresholds are low enough to game with a handful of planted trajectories.
