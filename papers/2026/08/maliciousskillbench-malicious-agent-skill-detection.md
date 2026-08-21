# MaliciousSkillBench: A Comprehensive Benchmark for Malicious Agent Skill Detection

- **Paper:** https://arxiv.org/abs/2608.19901
- **Authors:** Yue Wang, Yi Liu, Gelei Deng, Ying Zhang, Yuekang Li, Zhenyu Chen, Leo Zhang
- **Published:** 2026-08-20
- **Code:** Not listed in abstract

---

## Problem Statement (plain English)

Agent Skills are reusable instruction packages (prompts + scripts + config) that LLM agents load to gain capabilities. That same distribution channel is a perfect malware delivery mechanism: a "skill" can carry a malicious script that runs with the agent's privileges. Existing malicious-skill datasets are fragmented — scattered across sources, formats, and evidence standards — so nobody can reliably measure how good (or bad) detection is. This paper builds a unified benchmark and shows current detectors are much weaker than they look.

## Methodology (technical)

- Consolidated 13 public sources; 11 contributed core malicious artifacts; deduplicated 8,414 raw malicious records → 7,539 normalized-unique identities → 4,588 operational structural families.
- After conservative cross-label conflict exclusion: primary benchmark of 9,740 Skills (7,505 malicious, 2,235 benign).
- Harmonized 11 attack categories for 4,983 malicious identities with source-native mapping support.
- Evaluated three learned text detectors (incl. TF-IDF SVM) and three off-the-shelf Skill scanners under three evaluation regimes: Random split, Structural-disjoint, and Source-Disjoint (held-out sources).

## Key Results (with numbers)

- Learned detectors: 0.882–0.932 Random Macro-F1, but only 0.653–0.665 under Source-Disjoint evaluation — i.e., they memorize source-specific patterns.
- Best word TF-IDF SVM: 0.932 / 0.916 / 0.665 on Random / structural-disjoint / Source-Disjoint, with 95.6% malicious recall but 62.4% benign false-positive rate on held-out sources.
- Off-the-shelf scanners: reduce false positives only at the cost of sharply lower malicious recall — no tool operates in a satisfactory regime.

## What's Novel

- First consolidated, deduplicated, cross-source benchmark for malicious Agent Skill detection at this scale.
- Systematic demonstration that Random-split evaluation massively overstates detector performance (Source-Disjoint gap of ~0.27 F1).
- Joint measurement of attack detection AND benign over-flagging — the tradeoff that matters operationally.

## My Connection (to Manny's work)

This is the eval infrastructure for the exact supply chain Manny worries about: agent skills/MCP-style packages are the new npm. If a client adopts agent skills, this benchmark's Source-Disjoint finding says off-the-shelf scanning will miss novel malicious skills or flag a majority of benign ones — so detection must be layered (behavioral sandboxing + allowlists), not just content filtering. It also gives concrete numbers to cite when arguing that "we scan skills" is not a control.

## What I Learned (plain English)

- Agent skill marketplaces are a fresh malware channel that current detectors handle poorly.
- Benchmark design matters: split by source, not randomly, or you'll fool yourself.
- Any agent-skill deployment needs defense-in-depth: the scan is a tripwire, not a gate.
