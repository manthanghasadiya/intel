# COPA: Continual Preference Optimization for Adaptive Prompt Injection Defense

- **Paper:** https://arxiv.org/abs/2608.19982
- **Authors:** Roshan Sood, Onat Gungor, Tajana Rosing
- **Institution:** UC San Diego (Tajana Rosing — UCSD CSE)
- **Published:** Announced Aug 21, 2026 (v1 submitted Aug 20, 2026)

## Problem Statement (plain English)

Prompt injection is an arms race: attackers keep inventing new ways to smuggle instructions into model inputs, but most defenses are static — fixed alignment objectives or filters that go stale the moment attack strategies evolve. Existing "lifelong alignment" methods adapt to shifting user preferences, but not to *adaptive adversaries* that deliberately evolve to exploit weaknesses in whatever defense was just learned. The result: deployed agents slowly lose their immunity to both old and new attack classes.

## Methodology (technical)

- Frames prompt-injection defense as a **lifelong learning problem**: the model must keep learning from new attacks without catastrophic forgetting of defenses against older ones.
- **Continual preference optimization**: incrementally incorporates feedback from newly observed attacks via **GRPO-based optimization** (group-relative policy optimization, the RL recipe behind DeepSeek-style reasoning training).
- **Margin-weighted experience replay** retains and replays past attack examples — weighted by how close they are to being forgotten — to preserve defenses against prior attack classes while absorbing new ones.
- Evaluated on **lifelong prompt-injection attack streams** against state-of-the-art static defenses.

## Key Results (with numbers)

- **Up to 6.3x reduction in attack success rate**; **4.4x average reduction** across lifelong attack streams vs SOTA defenses.
- Maintains general-purpose model capabilities (no catastrophic forgetting of utility while learning new defenses).

## What's Novel

First treatment of prompt-injection defense as a continual-learning problem with an explicitly *adaptive adversary* model. Combines GRPO-based online adaptation with margin-weighted replay — a mechanism borrowed from continual learning — instead of one-shot alignment or static filtering. Directly addresses the "defense staleness" gap that fixed guardrails keep hitting.

## My Connection (to Manny's work)

Manny's agent stacks and any LLM-security product he evaluates currently rely on static guardrails that go stale as injection techniques evolve. COPA is a concrete, numbers-backed argument for building self-updating defenses (continual fine-tuning + replay) into agent runtimes, and its margin-weighted replay is a cheap pattern to steal for keeping red-team knowledge bases relevant.

## What I Learned (plain English)

- Prompt-injection defense is not a one-time alignment problem — it's a lifelong learning problem against an adversary that adapts to you.
- GRPO-style optimization can be pointed at attack data directly, and experience replay with forgetting-aware weights stops new defenses from erasing old ones.
- "Up to 6.3x / 4.4x average ASR reduction" — continual adaptation beats static SOTA by a wide margin once attacks evolve.
