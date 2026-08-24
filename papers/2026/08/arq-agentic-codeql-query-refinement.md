# ARQ: Agentic CodeQL Query Refinement for C/C++ Vulnerability Detection

- **Link:** https://arxiv.org/abs/2608.20637
- **Authors:** Chunyi Wang, Yunfei Ke, Junfeng Yang, Yun-Yun Tsai, Penghui Li
- **Institution:** Columbia University (Junfeng Yang) + IBM Research (Yun-Yun Tsai, Penghui Li)
- **Published:** Aug 24, 2026 (submitted Aug 21, 2026)
- **Code:** Not released (per abstract page)

## Problem Statement (plain English)

Query-based static analyzers like CodeQL find vulnerabilities by matching handwritten patterns against source code. The problem: these queries are full of false positives (flagging safe code) and false negatives (missing real bugs). Writing and tuning them is expert, tedious manual work — so coverage stays patchy.

## Methodology (technical)

- Key insight: **synthesize a program, run it, and compare execution against the query's verdict.** If a program is genuinely vulnerable but the query stays silent → the query has a false-negative weakness; if the program is safe but the query fires anyway → false-positive weakness. Execution is the ground truth that exposes where the query is wrong.
- ARQ then runs an LLM-based refinement loop: generate programs that expose weaknesses, feed the disagreement evidence back, and iteratively refine the CodeQL query until the disagreement is resolved.

## Key Results (with numbers)

- The abstract establishes the FP/FN weakness-discovery mechanism and the agentic refinement loop; headline numeric results (improvements over baseline CodeQL queries) are detailed in the paper body.

## What's Novel

- Uses *execution-grounded evidence* (synthesized programs + actual behavior) instead of relying on the LLM to judge code quality directly — the model proposes, execution disposes.
- Automates the query-authoring workflow that has stayed manual in the CodeQL ecosystem for years.

## My Connection (to Manny's work)

Directly useful for red-team/blue-team tooling: the same "synthesize → execute → compare verdicts → refine" loop applies to any signature/query/rule authoring task (YARA, Semgrep, SIEM rules), not just CodeQL. Also a good pattern for agentic vulnerability research — let the agent generate evidence, not opinions.

## What I Learned (plain English)

LLM agents get much more reliable at security tasks when they're grounded in execution rather than judgment. The recipe — generate a discriminating test case, run it, and use the disagreement as the learning signal — is a template for trustworthy agentic analysis in general.
