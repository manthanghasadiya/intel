# Trojan Horse: Automated Repository Poisoning against Coding Agents

**Authors:** Xiao Yang, Jinchang Hou, Jiawei Chen, Zhonghou Lv, Junyuan Shang, Chenfu Bao
**Institution:** Not stated on the arXiv listing
**Date:** Submitted Oct 16, 2026 (announced Oct 19, 2026)
**Link:** https://arxiv.org/abs/2610.23109
**Code:** Not stated in the abstract

---

## Problem Statement (plain English)

Coding agents read repositories the way a new hire does: they open files, follow instructions, run tests, and edit code. That makes the repository itself an attack surface. Prior red-teaming work on this only tested hand-picked payload locations and fixed disguises, which tells you little about what a real attacker — or a real automated attack — could achieve. The paper asks: if you automate the choice of *where* to hide a payload and *how* to disguise it, how often does a coding agent actually take the bait, and can any current agent stop it?

## Methodology (technical)

The authors build **TrojanForge**, a three-stage framework:

1. **AST mapping.** It parses the repository into an abstract syntax tree and maps the agent's execution traces onto AST nodes, so it knows which code regions the agent actually reads, reasons about, and executes.
2. **Conspicuousness ranking.** A security-review agent scores candidate mount points and selects the least suspicious ones — comments, test fixtures, helper modules, config-adjacent code — rather than obvious entry points.
3. **Adversarial refinement.** It plays against the target agent, observes why a payload was caught, inverts those causes into a new disguise, and carries successful strategies across repositories as transferable experience.

They also introduce **TrojanBench**: 384 poisoned repositories spanning 6 domains and 9 programming languages, evaluated across 4 agent harnesses and 12 models, including commercial agents.

## Key Results (with numbers)

- **Over 60%** stable poisoning success rate across the benchmark.
- **Over 90%** backdoor injection rate.
- **384** poisoned repositories, **6** domains, **9** programming languages.
- Evaluated on **4** harnesses and **12** models.
- Commercial agents **failed to defend** against repository-level poisoning in the authors' evaluation.

## What's Novel

- First *automated* repository-poisoning framework rather than manual payload placement.
- Uses the agent's own execution trace as the signal for where a payload will be effective — the attack is optimized against the agent's actual behavior, not a static notion of "important file."
- Adversarial disguise refinement that transfers across repositories, turning a one-off trick into a reusable capability.
- A benchmark with enough harness/model diversity to make the "agents can't defend" claim meaningful.

## My Connection (to Manny's work)

This is the supply-chain side of agent security Manny already works on from the MCP angle. The practical takeaway: any coding agent with repository write access is a supply-chain participant, and repo content is untrusted input even when it is "our own" code. TrojanForge's AST-plus-trace ranking is also a useful offensive playbook for red-team engagements — and its mirror image (flag code regions the agent reads but a human reviewer never touched) is a cheap detection heuristic to prototype.

## What I Learned (plain English)

A coding agent cannot tell the difference between "code that is part of the project" and "code that was placed where the agent would look." Automating the placement makes the attack cheap and repeatable; more than half the time it works, and the backdoor lands in over 90% of attempts. The defense cannot be "review the diff" alone — it has to be provenance: who wrote this file, when, and did a human ever approve it.
