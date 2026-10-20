# SpanGuard: Decoupling Instruction Localization from Authorization for Lightweight IPI Defense

**Authors:** Yinxiang Wang, Zehua Ma, Kejiang Chen, Weiming Zhang, Han Fang | **Affiliation:** not listed on the arXiv record
**Published:** October 19, 2026 | **arXiv:** [2610.26013](https://arxiv.org/abs/2610.26013) (cs.CR)
**Status:** Under review, 15 pages, 4 figures | **Code:** not released

---

## Problem Statement

LLM-based guardrails work against indirect prompt injection (IPI) — but only if the guardrail model is roughly as capable as the model it is guarding. Shrink the guardrail to something you can actually afford to run on every tool response, and detection falls apart. This is the deployment trap: the defense that works is too expensive to run inline, and the defense you can afford doesn't work.

The paper asks the right diagnostic question: *why* do small models fail here? The answer is that an IPI guardrail is being asked to do two different jobs at once — **locate** the injected instruction somewhere inside a large heterogeneous tool output, and **decide** whether the user actually authorized it. A small model can do either in isolation; it cannot do both jointly.

## Methodology

**The decoupling.** Split the task into two stages with different capability requirements:

1. **Localization (attack-agnostic, unsupervised, no LLM).** Injected instructions are usually *semantically discontinuous* with the content around them — a wall of JSON product data that suddenly contains imperative English. The paper partitions the observation into local regions using lightweight unsupervised segmentation. No model, no training, nothing to attack.
2. **Authorization (small LLM, only on the localized region).** Once you have a candidate block, the question collapses from "is there an injection in this 40KB blob" to "did the user ask for this?" — a short-context judgment a small model handles well.

**Why this is the right decomposition.** The expensive part of the original problem was never the judgment; it was the search. By making the search structural rather than semantic, the guardrail's capability requirement drops by roughly a model tier while the security property is preserved.

**Block-level rather than whole-input judgment** also means a single malicious span cannot launder itself by sitting inside otherwise-legitimate content, and legitimate content cannot be flagged because one region looks odd.

## Key Results

| Metric | Result |
|---|---|
| Attack success rate | **0.32%** |
| Benign utility loss | negligible |
| Guardrail model size | effective with substantially smaller models than prior work |
| Degradation with smaller guardrails | markedly reduced vs. joint-detection baselines |

The 0.32% ASR is the headline, but the more important result is the *slope*: prior LLM-guardrail approaches degrade sharply as the guardrail model shrinks; this one degrades much more slowly. That is what makes it deployable — you can pick your guardrail size on a latency budget instead of on a security requirement.

## What's Novel

1. **Naming the joint-task problem.** The observation that small guardrails fail because of task *conjunction*, not task difficulty, is the insight the rest of the paper falls out of.
2. **Unsupervised localization.** Attack-agnostic segmentation means the localization stage has no learned component to poison, no training distribution to drift from, and no per-attack tuning.
3. **Capability reallocation instead of capability scaling.** The field's default move is "use a bigger guardrail." This says "change what you ask the small one to do."
4. **Block-level authorization.** Localized judgment is both cheaper and structurally more robust than global judgment.

## My Connection

This is the deployable version of IPI defense for MCP. The realistic MCP threat model is: a tool returns a large blob (a webpage, a PDF, a ticket description, a GitHub issue) and somewhere in it is an instruction. Running a frontier model as a guardrail on every tool response is not a product anyone ships — the latency and cost are prohibitive at agent-loop call volumes. SpanGuard's structure maps cleanly onto a tool-response firewall: segment the response, run a small classifier on candidate blocks, and only escalate blocks that pass localization.

The "semantically discontinuous" heuristic is also a useful detection signal in its own right — imperative mood inside a data payload is a cheap static flag, and you can ship that today as a logging rule before you ship the full guardrail.

## What I Learned

The reason "just use a bigger model" keeps being the answer to security problems is that we ask models to do too many things at once and then blame their size when they fail. SpanGuard's contribution is mostly an act of *task design*: it noticed that localization and authorization are different problems with wildly different capability requirements, and that only one of them needs a language model.

Practically: when a defense is too expensive, the fix is usually not a cheaper model — it's a better decomposition. Cheap structural filters in front of expensive semantic judgments is the same pattern as signature-then-sandbox, and it works here for the same reason: the cheap stage doesn't have to be smart, it only has to be *selective*.
