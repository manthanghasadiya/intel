# Open Security Benchmark: Towards Autonomous Enterprise Cyber Defense

- **Link:** https://arxiv.org/abs/2607.27288
- **Authors:** Gal Engelberg, Michael Arenzon, Leon Goldberg
- **Published:** Jul 29, 2026 (v1)
- **Code/Datasets:** public (harness + synthetic-organization datasets linked from the paper)

## Problem Statement (plain English)

Enterprises are moving toward autonomous cyber defense — agentic AI that builds situational awareness of the org's security state and reasons from it to assessments and actions. But there is no public, shared, queryable environment to evaluate such agents end-to-end: real enterprise environments are private, cross-vendor, and deeply correlated. The authors call this the **environment data gap**, and it means the field cannot yet answer whether an agent's security answer should be trusted.

## Methodology (technical)

- **Open Security Benchmark (OSB)** surfaces a curated, frozen enterprise environment — an immutable snapshot of security state — and evaluates posture investigation in two modalities:
  1. **Text-to-SQL** over a relational snapshot of the environment
  2. **Native vendor APIs** over a served instance of the same environment
- Freezing the environment pins the target state, anchoring answers to closed-form ground truth.
- Five components: data layer, task/evaluation-set layer, multi-dimensional scoring layer, minimal auditable harness, and a bring-your-own path supporting both public comparison and private tenant evaluation.
- Instantiated with two identity-security packs and a family of synthetic-organization datasets spanning multiple scales.

## Key Results (with numbers)

- Framework ships with 2 identity-security task packs and multi-scale synthetic-organization environment datasets.
- Two evaluation modalities (text-to-SQL and native API) over the same frozen environment.
- Designed to extend from assessment toward remediation (later defense stages).

## What's Novel

First framework to make enterprise security-state data a shared, queryable benchmark target — solving the environment data gap with frozen snapshots and closed-form ground truth instead of relying on live (and private) enterprise environments. Bring-your-own substrate means orgs can evaluate agents against their own environment without publishing it.

## My Connection (Manny's work)

Defense-side benchmarking matters for offense: OSB-style frozen environments are exactly the kind of realistic, queryable terrain an offensive agent should train and be scored on — and the mirror of StealthBench. If defenders get trustworthy autonomous posture agents, the bar for evading detection-adjacent tooling rises.

## What I Learned (plain English)

You can't trust an agent you can't test, and you can't test a defensive agent without realistic environment data — which almost nobody publishes. Freezing a snapshot of the environment and scoring answers against closed-form ground truth is a clean way to make "trustworthiness" measurable. The same trick applies to evaluating red-team agents: pin the target, freeze the ground truth, score the run.
