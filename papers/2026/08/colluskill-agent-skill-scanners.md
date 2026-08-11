# ColluSkill: Adversarial Cross-Skill Composition for Evading Agent Skill Scanners

- **Paper:** [arXiv:2608.09732](https://arxiv.org/abs/2608.09732)
- **Authors:** Puyu Zeng, Simeng Qin, Jingzhi Li, Ju Jia, Zheli Liu, Xiaojun Jia
- **Institution:** Nankai University (based on the author team's prior affiliations; not stated on the arXiv page)
- **Published:** 2026-08-10 (v1) — 9 pages, 3 figures, 4 tables
- **Subjects:** cs.CR, cs.AI
- **Code:** Not listed on the arXiv page.

## Problem Statement (plain English)

Agent skills (packaged capabilities an agent can install) are a growing attack surface, and current defenses scan skills *individually*. That leaves a blind spot: several skills can each look perfectly benign on their own but, when executed in sequence, collectively perform a harmful workflow. The paper builds an attack that exploits exactly this gap, then proposes a defense that scans skills in context.

## Methodology (technical)

- **ColluSkill (attack):** decomposes one malicious intent into interdependent sub-payloads, each embedded in a separately packaged, locally-plausible skill. The harmful behavior emerges only from *ordered composition* of these skills via:
  - contextual dependencies (each skill's behavior depends on prior skills' outputs),
  - artifact passing (files/data handed between skills),
  - execution handoffs (one skill invokes the next).
  - LLM-based chain planning + scanner-feedback refinement keep the chain's attack semantics while minimizing suspicious signals in each individual sub-skill.
- **ChainGuard (defense):** a context-aware skill-chain scanner that jointly analyzes a candidate skill against the skills already installed in the agent environment. It reconstructs cross-skill dependencies, artifact flows, capability compositions, and downstream behaviors to flag risks that only appear at the workflow level.

## Key Results (with numbers)

- Empirical study of existing skill scanners shows they inspect individual skills, leaving cross-skill composition risks unexamined.
- ColluSkill chains evade per-skill scanners while preserving chain-level attack semantics.
- ChainGuard catches risks that emerge only at workflow level (numbers/rates in the paper's 4 tables).

## What's Novel

Frames agent-skill security as a *compositional* problem rather than a per-artifact one, and ships both ends: a working collusive multi-skill attack framework and a defense that reasons about installed-skill context. The scanner-feedback refinement loop (attacker iterates against the scanner) is a practical evasion engineering detail most threat papers omit.

## My Connection (to Manny's work)

- Landed the same day as the GhostSplice MCP disclosure — same root cause, different layer: splitting malicious intent across boundaries (tool channels vs. skill packages) defeats single-component scanners.
- Direct checklist item for any agent harness Manny audits: does the security review look at the *composition* of installed skills/tools, or just each artifact in isolation?
- ChainGuard's "joint analysis against installed environment" is the pattern to adopt for MCP server vetting.

## What I Learned (plain English)

A malicious capability doesn't have to live in any single file — it can be spread across many innocent-looking pieces that only become dangerous in a specific order. Security tooling that scores artifacts independently will miss this class entirely; context-aware, cross-artifact analysis is the only thing that catches it.
