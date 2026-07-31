# ThreatForest: Multi-Agent Attack Tree Generation with Pluggable TTP Framework Mapping

- **Link:** https://arxiv.org/abs/2607.27528
- **Authors:** Cristian Leo, Anton Dykyi, Danny Cortegaca, Daniel Begimher, Prakash Jha
- **Published:** Jul 29, 2026 (v1)

## Problem Statement (plain English)

Threat modeling of cloud-native architectures is slow and requires scarce security expertise. ThreatForest is an end-to-end system that takes a source-code repository and produces structured attack trees — with each attack step mapped to adversary TTPs from MITRE ATT&CK, CAPEC, and cloud-specific threat matrices — plus evidence-based mitigations.

## Methodology (technical)

- Multi-stage agent pipeline orchestrated as a directed graph:
  1. Repository analysis
  2. Context refinement
  3. Threat generation
  4. Parallel attack-tree construction with TTP mapping and mitigation synthesis
  5. Report generation
- Deterministic verification gates, bounded retries, and three human-in-the-loop validation points.
- A domain-specific sentence-transformer maps each attack step to candidate techniques by cosine similarity.
- Evaluated across 7 application domains on a 16-dimension rubric, scored by independent LLM raters with an adversarial verification pass and expert review.

## Key Results (with numbers)

- Panel-measured quality: **0.63–0.68 (0–1 scale)** for threat statements, attack trees, and mitigations.
- **0.29 for embedding-only TTP mapping** — the embedding stage, not the multi-agent pipeline, is the dominant accuracy bottleneck.
- The gap is stable across all seven domains.
- A controlled single-call baseline on the same model more than doubles mapping defensibility — pinning the limitation on the embedding encoder, not the agent design.

## What's Novel

First end-to-end system turning a repository into TTP-mapped attack trees with evidence-based mitigations across multiple adversary frameworks — and a reusable benchmarking framework for such systems. The explicit isolation of the embedding encoder as the binding constraint is a rare, honest ablation result.

## My Connection (Manny's work)

Attack-tree generation is the reconnaissance/planning half of an offensive agent loop. ThreatForest's architecture (multi-agent stages + deterministic gates + HITL checkpoints) is a reference design for agent pipelines where hallucination is unacceptable — and its TTP-mapping weakness is a concrete place to plug in better retrieval/embedding before trusting agent-generated attack plans.

## What I Learned (plain English)

When a multi-agent system fails, the bottleneck is often a single boring component — here, a sentence-transformer doing similarity matching — not the fancy orchestration. Agentic pipelines should be profiled the same way as code: find the component that caps accuracy and fix it directly. Also: LLM-judge panels with adversarial verification passes are becoming the standard way to score unstructured agent output.
