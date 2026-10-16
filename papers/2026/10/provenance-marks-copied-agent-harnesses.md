# What Survives Self-Evolution: Provenance Marks in Copied Agent Harnesses

**Authors:** Jieyu Li, Jingya Li, Sheng Li, Xinpeng Zhang, Zhenxing Qian
**Date:** Submitted Oct 15, 2026
**Link:** https://arxiv.org/abs/2610.21458
**Code:** Not stated in abstract

---

## Problem Statement (plain English)

An "agent harness" is the prompt + tool code wrapped around a base model. Harnesses leak: the paper cites Claude Code's ~512,000 lines being mirrored within hours of a packaging error. A plagiarist can take a leaked harness, run it through a published self-evolver (an LLM that rewrites and improves the code), and ship the result as their own. The authors define this as a **covert rewriting attack** — nobody explicitly targets the provenance watermark, yet the evolver's rewrite loop can quietly erase it.

## Methodology (technical)

- First systematic **measurement of provenance marks under covert rewriting**.
- Test matrix: **three evolvers × five proposers × two environments**.
- Published watermarking schemes tested include **KGW** (embedded in the prompt) and **AgentMark** (as released).
- Then they design a **new mark** built from the *order of near-tied entries* in a table of "where objects are found," constructed from public games — so deleting the table kills the harness's success, but rewrites that keep its rows preserve the ordering, and a customer can read the key from the agent's observable actions alone.

## Key Results (with numbers)

- Published marks do not survive: **KGW in the prompt survives GEPA's rewrites in 0 of 12 copies**; **AgentMark as released survives in 5 of 12**.
- The **proposer's habits** decide mark survival, and they vary by **two orders of magnitude** across models.
- Selection protects a mark only when each decision sees enough games.
- Their new ordering-based mark is **detected in 62 of 76 copies** whose lineage rewrote its file.
- A **scheme-aware plagiarist** reshuffles the near-ties at no measurable cost; asking the proposer for more general code removes the key in **75–100%** of rewrites.

## What's Novel

- Formalizes the *covert rewriting attack* on agent-harness provenance — a threat model nobody had measured.
- Quantifies that LLM self-evolution, not an adversary, is often what erases a watermark.
- Contributes a watermark keyed to behavioural ordering that is robust to ordinary rewrites yet honestly reports its own limits (scheme-aware and generalization attacks defeat it).

## My Connection (to Manny's work)

Directly relevant to agent supply-chain and IP/attribution questions Manny tracks. If leaked harnesses can be laundered through public self-evolvers, then provenance of agent code and prompts is a security and legal surface — the same "supply chain" theme as malicious MCP skills and poisoned tool repos. Also a useful mental model: any defence that a model's own optimization loop can silently rewrite should be assumed weak.

## What I Learned (plain English)

Watermarking agent code in the prompt is fragile because the tool that's supposed to improve the code also destroys the mark. Robust provenance has to hide in *behaviour* (the order the agent does things), not in text the model can paraphrase away. And even then, a smart plagiarist who knows your scheme can strip it — so treat provenance marks as a speed bump, not a lock.
