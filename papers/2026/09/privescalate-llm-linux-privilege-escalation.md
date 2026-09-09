# PrivEscalate: Measuring and Augmenting the Threat of LLM-Automated Linux Privilege Escalation

- **Link:** https://arxiv.org/abs/2609.09087
- **Authors:** Yixuan Liu, Zilong Zhen, Yin Wu, Yi Li
- **Institution:** Not stated in abstract
- **Published:** 2026-09-08 (cs.CR)
- **Code:** Not linked in abstract

## Problem Statement (plain English)

LLM agents are increasingly used for the offensive part of the kill chain, but nobody has measured how good they actually are at the crucial local step between "initial access" and "game over": Linux privilege escalation. Prior evaluations used fewer than 15 scenarios — far too small to rank models or architectures with any confidence. We need a large, executable, reproducible way to answer: which models/agents can actually escalate privileges on a real Linux box, and what makes them succeed or fail?

## Methodology (technical)

- **PrivEscalate benchmark:** 531 Dockerized privilege-escalation scenarios spanning 14 sub-categories, plus 329 parameterized variants that inject environmental distractors to test sensitivity (e.g., config rotation, noise).
- **Evaluation matrix:** six LLMs × three agent architectures, scored under executable verification (does the exploit actually work in the container, not just does the model say the right thing).
- **PrivEscAgent:** a domain-specialized wrapper around a generic ReAct agent adding three deterministic stages — enumeration, vulnerability-category matching, and step planning — without modifying the underlying LLM.

## Key Results (with numbers)

- Model capability is **heterogeneous across vulnerability classes**: no single model dominates across the high-prevalence classes → single-model risk assessments are misleading.
- LLM success is **sensitive to environmental perturbation**: configuration rotation disrupts some exploit attempts but does not eliminate measured risk.
- **Agent architecture materially changes success rates and reorders model rankings**, though the magnitude is model-dependent.
- PrivEscAgent **improves over prior Linux privilege-escalation agent baselines** without any LLM modification — i.e., the harness matters as much as the brain.

## What's Novel

First large-scale (500+ scenario), executable-verification benchmark for LLM-automated Linux privesc; explicit measurement that agent architecture can reorder model rankings; a wrapper-only augmentation that lifts performance — evidence that harness design is a first-class variable in offensive-agent capability.

## My Connection (to Manny's work)

Directly relevant to Intrud3r's offensive-agent stack: 531 privesc scenarios give a ready-made testbed for validating any agent harness Manny fields post-exploitation, and PrivEscAgent's "enumeration → category match → plan" wrapper is a pattern worth stealing for reliability. The distractor-sensitivity result is also a defensive lever: environment rotation degrades autonomous attackers.

## What I Learned (plain English)

- Small benchmarks lie: with <15 scenarios you cannot rank offensive agents; you need hundreds of executable cases.
- The agent's wrapper/harness can matter more than the model — a good deterministic scaffold beats a bigger brain with none.
- Rotating configuration/environment details genuinely slows LLM attackers, but only reduces — never removes — the risk.
