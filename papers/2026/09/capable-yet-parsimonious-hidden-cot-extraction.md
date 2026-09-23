# Capable yet Parsimonious: Extracting and Characterizing Hidden Chain-of-Thought in Frontier Models

**Authors:** Xiaoyu Luo, Tao Ren, Wenrui Yu, Xiao Li, Qiongxiu Li, Johannes Bjerva
**Institution:** See paper (Aalborg University and collaborators)
**Date:** September 22, 2026
**Link:** https://arxiv.org/abs/2609.26637

---

## Problem Statement (plain English)

Frontier closed-source models attribute their capability gains to better "reasoning," but you can't verify that because their raw chain-of-thought (CoT) traces are hidden. We can only see the final answer. This paper asks: can you *force* a closed model to show its reasoning using a normal API feature, and if so, how does that hidden reasoning actually work — is it genuine, and how do different models organize it?

## Methodology (technical)

- **Extraction technique:** register a **simple custom tool via a standard API feature** and induce the frontier model to externalize its intermediate reasoning (the tool-call/emit path becomes a channel for the model to write out its thinking).
- **Validity check first:** because externalized traces could be post-hoc rationalization, they compare against **native CoT** on open-source models and check whether the extracted reasoning *actually helps*. If extracted CoT matches native-CoT performance and beats no-reasoning baselines, it's carrying real signal.
- **Evaluation domains:** competition mathematics, science, and code generation.
- **Characterization axes:** token efficiency, reasoning-step types, and induced reasoning trees — to compare how models externalize, compress, and organize intermediate reasoning.
- **Models:** open-source baselines plus closed-source frontier models including **GPT-6 Astra**.

## Key Results (with numbers)

- Extracted reasoning **matches native CoT performance** and **substantially outperforms no-reasoning baselines** across math, science, and code.
- Systematic **inter-model differences** found in token efficiency, reasoning-step composition, and reasoning-tree structure.
- **GPT-6 Astra** exhibits **token-efficient "directed reasoning"**: it selects a correct trajectory earlier, resolves elementary steps internally, and externalizes only the crucial reasoning steps.
- Provides a **behavioral lens** on frontier reasoning beyond benchmark scores.

## What's Novel

- A **practical, standard-API extraction channel** for hidden CoT in closed models — no weight access needed.
- The methodological guard against post-hoc rationalization (validating extracted traces against native CoT and no-reasoning baselines) instead of assuming externalized text is honest thinking.
- A **structured characterization** (token efficiency / step types / reasoning trees) rather than a single accuracy number — showing models differ in *how* they think, not just how well.
- Introduces "directed reasoning" as an observed behavior for a frontier model (Astra).

## My Connection (to Manny's work)

Two threads. First, **defensive/interpretability**: a standard-API mechanism to pull reasoning out of a closed model is a transparency *and* an information-leak surface — the same channel can be abused to extract internal reasoning an operator may not want exposed, so it belongs in his model-behavior threat modeling. Second, **attack-craft**: understanding step types and compression (internal vs external reasoning) informs how to place prompt-injection and reasoning-derailment payloads where a model actually "thinks" — complementing the A2M trace-optimization work. The token-efficiency finding also matters for CoDoS-style cost attacks.

## What I Learned (plain English)

You can coax a locked model into showing its work by giving it a normal tool to "think out loud" into — and the reasoning it shows is real (it performs as well as the model's hidden native thinking). Once you can read that thinking, you find models differ a lot: GPT-6 Astra is efficient, solving easy sub-steps silently and only writing out the parts that matter. Practically, that means hidden reasoning isn't truly hidden if the API hands you a channel — a transparency feature that's also a leakage and manipulation surface.
