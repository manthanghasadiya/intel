# SkillTrace: Multi-Trace Provenance Auditing for LLM-Agent Skill Reuse

**Paper:** SkillTrace: Multi-Trace Provenance Auditing for LLM-Agent Skill Reuse
**Link:** https://arxiv.org/abs/2608.05204
**Authors:** Jialuo Chen, Minghe Wang, Lingqi Jiang, Jianan Ma, Xinhao Deng, Xiaohu Du, Ruixiao Lin, Yunhao Feng, Linkang Du, Jingyi Wang
**Institution:** Not stated in arXiv metadata
**Published:** August 5, 2026 (announced in Aug 7 new listings)
**Code:** Not listed in arXiv record

---

## Problem Statement (plain English)

Agent skills are becoming marketplace artifacts — packages of instructions, code, tools, and workflows that agents load and reuse. When someone "reuses" a skill (or copies it), how do you prove it? Ordinary code-clone detection fails because skill reuse evidence is spread across written instructions, code fragments, and operational structure — a reuser can preserve only one part and evade detection. If you can't audit reuse, you can't enforce licensing, provenance, or security review of what's flowing through your agent ecosystem.

## Methodology (technical)

- SKILLTRACE extracts **three provenance traces** from a skill: *Expression* (authored text), *Implementation* (code), and *Operational* (how it activates — represented as a **Skill Operational Graph (SOG)** capturing activation, procedure, and resource-flow structure).
- An LLM assists only the Operational-trace extraction, once at ingestion; at audit time, comparison is **deterministic** over cached traces.
- Each trace is calibrated against same-function strict negatives; the framework reports *which* trace supports a reuse decision.
- Benchmarked on SKILLTRACE-BENCH: 820 transformed reuse positives over 100 marketplace anchors + 751 negative controls.

## Key Results (with numbers)

- **AUROC 0.938, F1 0.898** on SKILLTRACE-BENCH.
- **Wild audit of 36,446 skills** shows trace-attributed evidence surfaces actionable reuse-review queues beyond what repository-level baselines catch.

## What's Novel

Frames skill reuse detection as a multi-trace provenance problem rather than single-modality clone detection, and introduces the Skill Operational Graph as the structural fingerprint. LLM use is confined to one-time ingestion — audits stay deterministic and reproducible.

## My Connection (to Manny's work)

Provenance auditing is the defense side of the skill-security coin (paired with today's AgentJailbreak attack paper). For Manny: if skills become the supply chain, skill provenance is the SBOM — this gives a concrete mechanism for "where did this skill come from and what is it actually doing" that red teams can also attack (can you launder a malicious skill past trace comparison?).

## What I Learned (plain English)

Reuse detection for agent skills is a different problem than code clone detection, and operational structure (what the skill actually activates) is the strongest evidence. Auditing can be made deterministic and cheap — LLMs only needed at ingestion — which makes provenance checking practical at marketplace scale.
