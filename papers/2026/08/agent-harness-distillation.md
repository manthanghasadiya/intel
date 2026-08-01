# Agent Harness Distillation: Inference-Time Harness Extraction and Exploitation in Autonomous Multi-Agent Systems

- **Paper:** https://arxiv.org/abs/2607.28147
- **Authors:** Yu Cui, Wuli Yang, Yirui Shi, Junhao Xia, Hui Jiang, Lei Gao, Chenfu Bao
- **Institution:** Not stated on abstract page
- **Published:** Jul 30, 2026 (arXiv cs.CR)
- **Code:** Not stated in abstract

## Problem Statement (plain English)

Autonomous multi-agent systems (AMAS) — think agent orchestrators like Hermes — rely on "inference-time harnesses": the orchestration logic that decides which agent does what, in what order, and how results are combined. Building these harnesses takes serious engineering and compute because they're iteratively optimized over a combinatorial search space while co-evolving with the underlying LLM. That makes the harness valuable intellectual property (IP). Prior work looked at IP leakage in static multi-agent systems with pre-configured architectures, but nobody had shown whether an attacker can steal the harness from a *dynamic* AMAS where harness behavior only emerges at inference time — through black-box interaction alone.

## Methodology (technical)

The authors introduce **Agent Harness Distillation (AHD)**, a two-stage framework:

1. **Pre-distillation stage:** AHD queries the target agent and infers its inference-time harness behaviors from the responses, constructing an initial approximation of the harness.
2. **Post-distillation stage:** AHD iteratively refines the initial harness, aligning it with the target's behavioral patterns until it reproduces comparable orchestration behavior.

The whole extraction is black-box — no access to weights, prompts, or internal state. The paper also formalizes harness extraction as a new security problem and builds an evaluation framework to quantify the risk, then evaluates against real-world AMAS running on multiple backbone LLMs.

## Key Results

- AHD successfully extracts inference-time harness capabilities from real-world AMAS across multiple backbone LLMs (per abstract).
- Experiments reveal **substantial IP leakage risks** in current systems.
- The authors additionally propose a **deception-based defense** that reduces extraction effectiveness (details in full paper).

## What's Novel

- Names and formalizes a previously unnamed attack class: inference-time harness extraction, distinct from model distillation or prompt extraction.
- First demonstration that dynamic, inference-emergent harness behavior can be cloned via black-box interaction.
- Introduces an evaluation framework and a deception-based countermeasure.

## My Connection (to Manny's work)

Manny runs agent stacks whose real value is orchestration — the harness — not just the underlying model. This paper says that value can be siphoned by anyone who can talk to the agent. Practical implications: treat harness behavior as sensitive, add behavioral watermarking or deception responses to high-value agent endpoints, and assume competitors (or attackers) can clone orchestration patterns from observed interactions. Also: AHD is exactly the kind of technique a red team would use to assess whether a client's agent IP is exfiltratable.

## What I Learned (plain English)

Your agent's "secret sauce" isn't in the code you can see — it's in the behavior the system produces, and behavior is observable by anyone with API access. If orchestration logic is valuable, it needs the same protection as weights: monitoring for probing, rate limiting, and deliberately injecting noise or deception so extracted clones are degraded.
