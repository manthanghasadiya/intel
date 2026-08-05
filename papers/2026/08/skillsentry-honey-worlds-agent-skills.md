# SkillSentry: Adaptive Honey Worlds for Dynamic Safety Testing of Agent Skills

- **Paper:** https://arxiv.org/abs/2608.03485
- **Authors:** Nizhang Li, Zonghao Ying, Xiangfan Wu, Zonglei Jing, Xixun Lin, Hao Zhang, Wenxin Zhang, Jiaye Lin, Quanchen Zou, Xiangzheng Zhang
- **Institution:** (multi-affiliation; overlaps with the SkillJack group)
- **Published:** Aug 4, 2026
- **Code:** https://github.com/nizhangli062-jpg/SkillSentry-Adaptive-Honey-Worlds-for-Dynamic-Safety-Testing-of-Agent-Skills

## Problem Statement (plain English)

A skill (the reusable capability files agents load) can look perfectly benign under static inspection and only turn malicious under specific runtime conditions — particular environment states, resources, or interaction histories. Existing skill scanners use static analysis, rule lists, or one-shot semantic judgments, so they can't elicit or attribute this *conditional* bad behavior. SkillSentry is a dynamic testing framework: it drops the skill into a simulated "honey world" and watches what it actually does.

## Methodology (technical)

1. **Capability inference** — infer the skill's intended capability boundary from its description/code.
2. **Honey world construction** — build an LLM-simulated environment with controlled decoy resources (fake credentials, files, network endpoints) designed to tempt conditional behavior.
3. **Adaptive task generation** — iteratively generate tasks that explore the skill's behavioral states, probing for the conditions that trigger malice.
4. **Attribution** — compare skill-enabled trajectories against matched no-skill executions; ground suspicious behavior in source code and verified execution traces before declaring it malicious.

Evaluated against seven scanner configurations (static + semantic baselines), including a semantics-preserving evasion setting where the malicious skill is rewritten to dodge one-shot judgments.

## Key Results (with numbers)

- **Standard benchmarks:** 99.50% Recall, 96.26% average F1.
- **Under semantics-preserving evasion:** 92.95% average F1 vs 80.07% for the strongest baseline.
- Execution-trace grounding gives it attribution (which code path, which environment state) that static scanners lack.

## What's Novel

Moves skill safety from inspection to *interaction*: instead of asking "does this look malicious?", it asks "under what conditions does this become malicious?" — then constructs those conditions. The decoy-resource honey world plus adaptive task generation plus trace-grounded attribution is a new combination, and it degrades gracefully against evasion, where static approaches collapse.

## My Connection (to Manny's work)

Directly the defense counterpart to SkillJack, and a template for MCP-server / agent-skill vetting. If Manny evaluates third-party MCP servers or skills before wiring them into an agent, SkillSentry's pattern (simulated env + decoys + differential execution) is the architecture to crib: run untrusted skills against fake credentials in a sandbox and compare behavior against a no-skill baseline. Also a candidate to run as CI for skill registries.

## What I Learned (plain English)

Static analysis of agent skills has a ceiling: malicious behavior can be gated on runtime state, so the only reliable way to find it is to *execute* the skill in an environment engineered to trigger it, and to attribute findings by diffing against a clean run. "Honey world" testing — giving the skill fake treasure and watching whether it steals it — is the agent-era equivalent of sandboxing executables, and it holds up against evasion much better than scanning.
