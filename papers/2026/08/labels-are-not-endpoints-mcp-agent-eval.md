# Labels Are Not Endpoints: Treatment Leakage and Construct Validity in MCP Agent Security Evaluation

- **Paper:** [arXiv:2608.12880](https://arxiv.org/abs/2608.12880)
- **Authors:** Rana Muhammad Ahmed, Sabahat Abbas
- **Institution:** Not stated on arXiv listing
- **Published:** 2026-08-13
- **Code:** Not listed

## Problem Statement (plain English)

Security benchmarks for tool-using (MCP-style) agents produce numbers like "X% of attacks succeeded." But those numbers are only meaningful if the label attached to each test actually reflects what the agent did. This paper audits a real, preserved MCP agent security evaluation campaign and shows the labels were contaminated: the grading logic used *treatment metadata* (which experimental condition a test belonged to) to decide whether an action counted as an attack success. That means the same fixed behavior could flip classification just by relabeling which treatment it came from — the benchmark's headline numbers were an artifact of the grader, not the agents.

## Methodology (technical)

- Traced a preserved campaign's full execution record: **10,200 execution rows → 180 model-bound requests → 45 semantic requests → 15 observable stimuli**.
- Reconstructed the evaluation design: two schema treatments were delivered, but the planned external payload-family corpus was *not* — an incomplete design whose grader still gated `ATTACK_SUCCESS` on treatment metadata.
- Ran a **treatment-blind reconstruction**: re-graded the same execution rows without access to treatment labels.
- Built a **locked v2 census** (a frozen, re-derived label set) and ran a **dual-reviewer blinded concordance review** of all 96 requests deemed structurally interpretable by the locked v2 codebook.

## Key Results (with numbers)

- **58 historical `ATTACK_SUCCESS` / `HIJACK_ATTEMPT` labels corrected** to authorized benign completions under treatment-blind grading.
- **Three verified protected-data transfers preserved** plus one separate unauthorized-forwarding case — the reconstruction kept the real positives.
- **Locked v2 census contains exactly zero `ATTACK_SUCCESS` records**; the forwarding case remains a `HIJACK_ATTEMPT` only at a semantic boundary concerning objective completion.
- Dual-reviewer consensus classes were **identical between reviewers but differed from the locked codebook** — even a cleaned label set disagreed with the frozen one.

## What's Novel

The first published audit-style demonstration of **treatment leakage in agent security evaluation**: grading conditioned on experimental metadata rather than behavior. It reframes the core validity question — "labels are a construct, not an endpoint" — and provides a reproducible audit methodology (row-tracing, treatment-blind re-grading, locked census, blinded concordance) for any agent security benchmark.

## My Connection (Manny's work)

Every eval Manny ships should survive this audit: graders must be treatment-blind, execution rows must trace to model-bound requests, and "attack success" must be defined behaviorally (what the agent did) rather than by which experimental arm the test came from. The 58 corrected labels are a warning that headline agent-security numbers in the MCP space may overstate risk — or misstate which behaviors are actually unsafe.

## What I Learned (plain English)

Benchmark labels are not ground truth — they are the output of a grading pipeline that can itself be biased. Before trusting any "attack success rate" for tool-using agents, ask: did the grader know which test condition it was grading, and would the same behavior get the same label if it appeared in a different condition? If not, the numbers are measuring the grader, not the agent.
