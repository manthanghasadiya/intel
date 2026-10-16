# Language Is a Security Surface for Agent Guards

**Authors:** Hasan Demirkiran, Jens Ernstberger
**Date:** Submitted Oct 15, 2026
**Link:** https://arxiv.org/abs/2610.22244
**Code / Data:** Benchmark released; all 489k verdicts released

---

## Problem Statement (plain English)

Step-level "guards" sit in front of an agent and decide, before each tool call, whether the next action is safe given the user's request and what's happened so far. Users write requests in many languages, but guard prompts and the benchmarks guards are trained and evaluated on are **English only**. So a natural question: does a guard's verdict change if the same request is simply written in another language? The paper shows the answer is yes — language is itself a security surface.

## Methodology (technical)

- Built **LinguaGuardBench** from the **1,220 labelled steps** of TS-Bench AgentDojo-Traj.
- Translated **only the user instruction** into German, French, Turkish, Swahili and Amharic, keeping every tool call, observation and label **byte-identical** — so any verdict change is attributable to language alone.
- Controls: **six English rewordings** plus a whitespace placebo; managed guards run three times to measure their own noise floor.
- Evaluated **19 guards**, spanning frontier APIs, 4B open models, and four purpose-built agent guards, reporting a placebo-adjusted flip rate.

## Key Results (with numbers)

- Most open, small and purpose-built guards change verdicts under translation **beyond their own noise floor**, and the effect **grows with language rarity**.
- Among the nine guards below 13B parameters, mean placebo-adjusted flip rate rises from **+1.5 pp on French to +4.4 pp on Amharic**.
- **Purpose-built guards are the most sensitive.**
- Direction follows size: open guards of **12–30B** block more benign steps (**up to +5.0 pp false positives**); every significant recall loss is in a guard **below 10B** (**up to −7.4 pp**).
- **Frontier guards shift by at most ~1 point**; the strongest, **Gemini 3.8 Flash, returns identical verdicts** with and without the request.
- Damage concentrates in a few instructions and falls on steps where the agent's reasoning restates the English request — consistent with guards reacting to **cross-channel inconsistency**. English paraphrase moves every guard less than translation does.

## What's Novel

- Isolates *language alone* as a cause of guard verdict changes via byte-identical controls — a clean causal design.
- Quantifies a "language tax" on guard reliability that scales with rarity, and locates it in small/purpose-built guards.
- Releases a benchmark and a large verdict corpus (489k verdicts) others can build on.

## My Connection (to Manny's work)

Guardrails are part of the agent defence stack Manny tests and writes about. This paper says: if your guard isn't language-robust, an attacker can bypass it by simply switching the request's language, and your benign users get false positives for the same reason. It's a concrete new bypass primitive (multilingual prompt injection / guard evasion) plus a benchmark to validate any guard you deploy.

## What I Learned (plain English)

Your AI safety filter is reading the request in English even when your user isn't. Ask the same thing in Amharic and a small guard may wave it through — or block a harmless request. Frontier models barely budge, but anything you train or fine-tune yourself needs to be tested multilingually, not just in English. Language is not a cosmetic layer on top of an agent; it's an input channel an attacker controls.
