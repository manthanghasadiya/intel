# Vero: Can AI Agents Build Formally Verified Software Repositories?

- **Paper:** [arXiv:2608.13522](https://arxiv.org/abs/2608.13522)
- **Authors:** Zhe Ye, Hantao Lou, Yuechun Sun, Peiyang Song, Zhengxu Yan, Timothe Kasriel, Qingyang Zhang, Kaiyu Yang, Soonho Kong, Jingxuan He, Dawn Song
- **Institution:** UC Berkeley (Sunblaze lab, Dawn Song group) + collaborators
- **Published:** 2026-08-13
- **Code:** https://github.com/sunblaze-ucb/vero

## Problem Statement (plain English)

AI agents write code, but nothing guarantees the code is correct. "Verified code generation" — an agent produces an implementation *plus* a machine-checked proof that it satisfies a formal specification — is the strongest path to trustworthy AI code. But existing benchmarks only test single functions, or give the agent the implementation and ask for a proof. Nobody has tested whether agents can make coherent implementation-and-proof choices across a real multi-module codebase. Vero is the first benchmark to measure that.

## Methodology (technical)

- **Benchmark:** 43 multi-module instances sourced from real-world repositories spanning Python, Dafny, Verus, and Coq, covering cryptographic protocols to distributed systems.
- **Format:** each instance is a multi-module Lean 4 repository with predetermined API interfaces, manually curated formal specifications, and reference implementations.
- **Modes:** proof-only and code-and-proof evaluation.
- **Audit mechanism:** agents may formally prove that a provided specification is unsatisfiable or reference code is incorrect — surfacing latent curation errors instead of penalizing agents for them.
- **Evaluation:** frontier coding-agent configurations with Lean toolchain access.

## Key Results (with numbers)

- The strongest agent fully solves only **27 of 43** instances.
- It closes **no specifications on the hardest repositories**.
- Conclusion: current agents fall short of repository-scale verified software synthesis; Vero is a concrete testbed for measuring progress.

## What's Novel

First benchmark for *joint* implementation + proof synthesis at repository level (not function level, not proof-only), with an audit mechanism that lets agents formally challenge the benchmark's own specs — a reliability feature no prior verified-coding benchmark has.

## My Connection (Manny's work)

For AI-red-team work this is a double-edged tool: (a) formal verification as the *ground truth* for "did the agent's code do what we asked" — far stronger than test-passing when assessing agentic coding security; (b) a benchmark for probing how far agent harnesses are from producing auditable, proof-backed artifacts. Also: 27/43 with zero closes on hard repos is the current ceiling — agents can't yet be trusted to self-certify security properties.

## What I Learned (plain English)

"AI-generated code" and "verified code" are currently two different worlds: even the best agent solves ~60% of repository-scale verified-coding tasks and fails completely on the hardest ones. Verified agent output is not around the corner, so red-team verification of agent-produced code remains manual — and Vero gives a fair way to measure when that changes.
