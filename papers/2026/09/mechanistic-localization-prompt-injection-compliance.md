# Where Do LLMs Decide to Break the Rules? Mechanistic Localization of Prompt Injection Compliance

**Authors:** Rui Wen, Jiayang Liu, Zeyu Yang, Jun Sakuma, Lu Sun
**Institution:** (Waseda University / RIKEN AIP area — per author groupings)
**Published:** 2026-09-29
**Link:** https://arxiv.org/abs/2609.37737
**Code:** Not stated in abstract

---

## Problem Statement (plain English)

When a prompt injection succeeds, the model abandons its assigned role and obeys the attacker. Prior work mostly measured *how often* this happens; this paper asks the mechanistic question: **where inside the network does the model actually decide to break the rules?** Knowing the answer matters for both defense (detect there) and interpretability (why detection at the surface fails).

## Methodology (technical)

The authors use **layer-by-layer causal activation patching** across **five models (4B to 32B parameters)**. For each layer, they patch activations to measure how much that layer causally controls whether the model complies with an injected instruction — as opposed to merely *containing* attack information. They then analyze the geometry of the compliance mechanism (its effective rank / subspace) and test whether the causally important layer is also the best place to *detect* attacks.

## Key Results (with numbers)

- **Dissociation:** attack information is **linearly decodable from the first layer**, yet its **causal leverage over behavior is negligible until a late-layer bottleneck in the final third** of the network. In other words: the information is present early, but the *decision* is made late.
- **Patching the bottleneck reverses compliance in 77–92% of cases.**
- **Compact subspace:** the compliance mechanism occupies a compact linear subspace — **rank-8 in the 4B and 14B models, scaling to rank-64 at 32B** — and is **architecturally stable across model families**.
- **Detection alignment:** the causal-peak layer is also the **representationally optimal site for detecting attacks**, outperforming early-layer classifiers that degrade under surface-level obfuscation such as **leetspeak substitution**.

## What's Novel

- Provides a **causal map** of injection compliance, not just correlational probing — showing the *information* and the *decision* live in different places.
- Quantifies the compliance mechanism as a **low-rank linear subspace** that grows with model size but stays compact.
- Shows the **causal layer and the best detection layer coincide**, which reconciles interpretability and practical detection: you should read the model where the decision is made, not where the text appears.

## My Connection (to Manny's work)

This is the explainer-ready version of "why your input filter is fooled by leetspeak." It gives a mechanistic reason to place detection/guards late (where the model commits) rather than early (where it merely sees). It also backs a compelling content angle: **injection defenses that read the prompt are reading the wrong layer** — the model has already linearly encoded the attack, but hasn't yet decided to obey it.

## What I Learned (plain English)

A model "sees" an injected instruction almost immediately — you can decode it from layer one — but it doesn't *choose* to obey until a narrow bottleneck in the last third of the network. Patch that bottleneck and the model stops complying; watch that bottleneck and you catch attacks that foolify surface tricks like leetspeak. The decision and the evidence are in different places.
