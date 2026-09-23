# A2M: Trace-Optimized Agent Hijacking in the MCP Ecosystem

**Authors:** Laizhen Li, Xuan Wang, Peicheng Zhao, Juanjuan Zhao, Kejiang Ye, Cheng-zhong Xu, Xitong Gao
**Institution:** Shenzhen Institute of Advanced Technology (SIAT), CAS; University of Macau (and collaborators)
**Date:** September 22, 2026
**Link:** https://arxiv.org/abs/2609.26761
**Code:** https://github.com/Lilaizhen/A2M

---

## Problem Statement (plain English)

MCP-based agents don't read tool documentation the way a human does — they pick tools by *semantic similarity* between the user's request and the tool's name/description. That means whoever controls a tool's metadata (name, description, parameter schema) and its runtime return values can influence which tool the agent calls and what it does next. This is a supply-chain attack surface: a malicious or compromised MCP server can steer an agent without ever exploiting a memory-safety bug. Prior MCP attacks mostly perturbed a single step; A2M asks whether you can *systematically and black-box* optimize both the tool's metadata (attraction) and the tool's outputs over time (manipulation) to reliably hijack agent behavior, and how well that transfers to models you can't query freely.

## Methodology (technical)

A2M ("Attraction-to-Manipulation") is a **two-stage, black-box** framework:

1. **Attraction phase** — optimize the malicious tool's metadata (name + description text) to maximize the probability that the agent's semantic matcher selects that tool for a target user query. No weights, no gradients into the model; purely query-based optimization against the tool-selection step.
2. **Manipulation phase** — once the tool is invoked, use the agent's **execution traces** (observations of reasoning + subsequent actions) as the optimization signal to refine *adversarial tool returns*. The tool's output strings are iteratively tuned to push the agent toward attacker-desired downstream actions.

Evaluation is on **LiveMCPBench** with GLM-4.6 as the optimize/eval model, across four scenarios: **Cognitive Denial of Service (CoDoS)**, **Information Exfiltration**, **Environment Integrity Compromise**, and **Reasoning Derailment**. They then test **transfer** to four other models without re-optimization.

## Key Results (with numbers)

- **Malicious tool invocation rate:** macro-average **93.6%** across four scenarios (GLM-4.6, optimized + evaluated on same model).
- **Cognitive Denial of Service:** weighted token cost inflated to **32.4×** the benign baseline.
- **Mean attack success rate:** **74.4%** across Information Exfiltration, Environment Integrity Compromise, and Reasoning Derailment.
- **Transfer (no re-optimization)** to four other models: macro-average invocation **63.6%**, token cost **2.7×**, success **24.5%**.
- Code publicly released.

## What's Novel

- Treats MCP hijacking as an **optimization problem over traces**, not a hand-crafted prompt-injection payload — closing the loop between metadata (attract) and output (manipulate).
- Fully **black-box**: works against closed models, and the transfer results quantify how much of the attack is generic vs. model-specific.
- Introduces **Cognitive Denial of Service** as a cost-amplification objective — weaponizing an agent's reasoning budget rather than its permissions.
- Demonstrates that metadata is a first-class, attacker-controlled input, strengthening the case for signed/verified tool manifests.

## My Connection (to Manny's work)

Directly relevant: Manny builds and red-teams MCP agents. A2M provides (a) a reproducible harness/codebase for testing tool-metadata poisoning against local agent stacks, (b) the metric set (invocation rate, token-cost inflation, scenario-specific success) to reuse in his own evals, and (c) the transfer numbers to argue why *per-model* defenses aren't sufficient — you need registry/registry-level vetting and runtime isolation. The CoDoS angle is also a fresh addition to his agent-DoS playbook alongside classic resource-exhaustion attacks.

## What I Learned (plain English)

An agent doesn't get hacked at the code level — it gets *persuaded* at the metadata and output level. If someone can name and describe a tool cleverly and then lie convincingly in its response, they can get your agent to call it 9 times out of 10 and do what they want nearly three-quarters of the time, even on models they've never touched. The defense isn't "make the model smarter"; it's controlling who can inject tool metadata and output into the agent, and sandboxing what a tool call can actually touch.
