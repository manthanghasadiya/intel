# Malicious Skill Files in Coding Agents (AgentJailbreak)

**Paper:** Towards a Risk Assessment of Malicious Skill Files in Coding Agents
**Link:** https://arxiv.org/abs/2608.05223
**Authors:** Rui Yang, Michael Fu, Kla Tantithamthavorn, Chetan Arora, Joey Chua
**Institution:** Not stated in arXiv metadata
**Published:** August 5, 2026 (announced in Aug 7 new listings)
**Code/Dataset:** https://github.com/awsm-research/AgentJailbreak

---

## Problem Statement (plain English)

Coding agents (Claude Code, Gemini CLI, Qwen Code) load "skills" — folders of instructions and scripts — to specialize behavior. That skill interface is also an attack surface: an attacker can hide a malicious shell command inside a natural-language skill file that looks completely benign. Nobody had quantified how often a coding agent would actually execute those hidden commands in real runs — or how often the agent even notices it's being asked to do something malicious.

## Methodology (technical)

1. **Adversarial skill synthesis:** Six LLMs across four model families transformed 471 real-world shell commands into benign-appearing skill files → benchmark of 2,826 skills mapped to 11 MITRE ATT&CK tactics.
2. **Reproducible evaluation pipeline:** run stratification + evidence anchoring + a refusal veto + a deterministic declared-intent override, scored by a three-judge LLM-as-a-judge panel, validated against a blind human gold standard (Cohen's kappa = 0.85).
3. **Large-scale characterization:** two enterprise-grade agents (Gemini CLI, Qwen Code) across 5,629 completed runs, measuring whether the hidden commands actually executed.

## Key Results (with numbers)

- **Gemini CLI executes the malicious skill in 95.5–96.1% of runs** (raw majority vote to declared-intent-corrected estimate, both within the human gold standard).
- **Qwen Code: 71.6–74.0%** execution rate.
- Execution rate was **nearly invariant to which LLM generated the skill** — the weakness is the interface, not the attacker's model.
- **Explicit safety recognition in only 1.99% of runs** — the agents almost never verbalize that something is off.

## What's Novel

First large-scale risk assessment of the skill-file interface specifically (not prompt injection generally), with a reusable benchmark + evaluation pipeline. The ~96% execution rate and ~2% safety-recognition rate are the headline numbers — they turn "skills are risky" from a vibe into a measurement.

## My Connection (to Manny's work)

This is a direct red-team playbook for coding-agent environments: generate skill files that pass as benign, drop them in a repo/skill dir, and measure execution. It also gives Manny the benchmark (2,826 skills, ATT&CK-mapped) to test *defenses* — skill allow-listing, provenance checks, sandboxed execution — against a known-bad corpus instead of hand-rolled examples.

## What I Learned (plain English)

Coding agents will run a well-crafted malicious skill almost every time, and they almost never say "this looks malicious." The skill interface — not the model — is the trust boundary, and right now it's wide open. Defenders should treat every skill directory like executable code: review, allow-list, sandbox.
