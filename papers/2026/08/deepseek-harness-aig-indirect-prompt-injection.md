# Security Assessment of DeepSeek Harness with A.I.G: Evaluating Resistance to Indirect Prompt Injection

- **Link:** https://arxiv.org/abs/2608.16393
- **Authors:** Zonghao Ying, Xiangfan Wu, Huiyu Wu, Xing Zheng, Huangsheng Cheng, Xiaorong Shi, Jing Guo
- **Institution:** Not stated in listing
- **Published:** Aug 17, 2026
- **Code:** Not listed

## Problem Statement (plain English)

DeepSeek Harness (DSH) is an agent framework, and nobody had systematically measured how resistant its agent loop is to *indirect prompt injection* — attacks where malicious instructions hide in content the agent reads (files, web pages, tool outputs). The paper builds a repeatable evaluation harness (A.I.G) that injects controlled taint through many channels and measures whether the agent acts on it.

## Methodology (technical)

- **A.I.G (AI-Infra-Guard)** constructs tests, delivers controlled taint, executes DSH, collects traces, and judges outcomes — preserving DSH's real agent loop, tool registry, model adapter, and session-event path.
- **Scale:** 14,560 controlled executions across **16 indirect-content channels**, text and file carrier modes, **35 payload objectives**, **12 attack methods**, plus one unmodified baseline.
- Source tools and sensitive sinks are local fixtures — attempted actions are recorded with no external side effects.
- Two judges: deterministic rule-based (**JudgeR/RuleJudge**) and semantic LLM-based (**JudgeL/LLMJudge**).

## Key Results (with numbers)

- **Strongest observed attack success rates:**
  - **25.5%** — hidden Unicode attack, file mode (JudgeR).
  - **17.0%** — fake-completion attack, text mode (JudgeL).
  - **16.0%** — (another attack configuration, per abstract; full details in paper).
- Attack success varies by judge, carrier mode, and attack method — no single channel dominates, but file-mode Unicode smuggling is the standout vector.

## What's Novel

A reproducible, high-volume (14.5k execution) IPI assessment of a specific open agent harness with dual deterministic/LLM judges — more systematic than typical one-off IPI demos, and directly actionable for hardening DeepSeek Harness deployments.

## My Connection (to Manny's work)

DeepSeek Harness is the tooling family Manny works with daily; these numbers are the baseline for its injection resistance: expect ~1 in 4 attacks to land via hidden-Unicode file content. Any DSH-based deployment Manny audits should treat file ingestion (docs, configs, repos) as the primary IPI ingress and add input normalization (Unicode sanitization) plus egress allowlists.

## What I Learned (plain English)

- Indirect prompt injection is not theoretical for DeepSeek Harness — real attack success rates of 17–25% across channels.
- File-mode attacks (especially hidden Unicode) beat text-mode attacks.
- LLM judges and rule judges disagree — evaluation methodology changes the measured success rate, so always report both.
- Normalize file content before the agent reads it; a Unicode-smuggled instruction survives otherwise.
