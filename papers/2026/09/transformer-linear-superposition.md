# Your Transformer Can Hold Two Thoughts at Once: Evidence of Linear Superposition in LLMs

**Authors:** Tikhonov, Korznikov, Mikhalchuk et al.
**Institution:** Not stated on the arXiv abstract page (verify affiliations in the PDF before citing)
**Published:** September 25, 2026
**Link:** https://arxiv.org/abs/2609.29845
**Code:** Not linked on the abstract page
**Subjects:** cs.LG / cs.CL (machine learning — check the listing for exact primary subject)

---

## Problem Statement (plain English)

Transformers are famously non-linear — attention, MLP activations, softmax; nothing about the machinery is supposed to add up neatly. This paper claims the opposite is true at a specific, testable place: if you hand the model the **arithmetic combination of two different text streams**, the model doesn't get confused — it produces the **arithmetic combination of the two would-be outputs**. In other words, the model can "hold two thoughts at once," and the blend is linear rather than scrambled.

## Methodology (technical)

- **Superposition Linearity Hypothesis (SLH)**: when inputs from two distinct text streams are **linearly combined at the input** (e.g. a weighted sum of two embeddings/prompts), the model outputs a **superposition of the individual next-token distributions** — the output is the same linear combination of the two single-stream outputs.
- They test whether this holds empirically, then ask **whether it is intrinsic to the architecture or an artifact of training**.
- They measure how superposition behaves **over the course of pretraining** (does it strengthen, weaken, or decay?).
- They test **interventions**: does lightweight fine-tuning change the degree of superposition?
- They build a **guided-decoding** procedure on top of the finding that generates **two coherent continuations from a single forward pass**.

## Key Results (with numbers)

- **Linear superposition holds**: linearly combining two input streams yields a superposition of the two next-token distributions. (The abstract reports the qualitative result; specific quantitative coefficients are in the PDF.)
- **Superposition is argued to be an intrinsic property of the transformer architecture**, not an emergent side-effect of training.
- **It decays over pretraining** — i.e. the property is strongest early and is progressively "used up"/masked as training proceeds.
- **Lightweight fine-tuning can substantially restore it**, suggesting the capability is dormant rather than destroyed.
- A **guided-decoding** method derived from the finding produces **two coherent continuations from one forward pass** — i.e. real compute savings, not just an interpretability curiosity.

## What's Novel

Most interpretability work locates "meaning" in individual features or directions; this work locates a **linear algebraic structure in how whole distributions combine** at the output, and then shows it is (a) architectural, (b) decaying with training, and (c) partially recoverable. The **"two thoughts, one forward pass"** decoding application is what turns it from a curiosity into something with an efficiency story — running two generation tracks for the price of one is exactly the kind of lever that matters once you're paying per token at agent scale.

## My Connection

Two angles. First, **efficiency**: a single forward pass emitting two coherent continuations is directly relevant to speculative decoding, parallel deliberation, and "cheap second opinion" patterns in agent harnesses — potentially a real cost lever for multi-hypothesis agent planning. Second, **control and monitoring**: if two streams pass linearly through the model, then an input that looks benign as a whole can still carry an adversarially chosen *component* — a structured mechanism for smuggling a second instruction channel that reviewers looking at "the prompt" won't parse. That's speculative on my part, but it's exactly the sort of thing worth red-teaming now, before it's a known trick.

## What I Learned (plain English)

A transformer isn't just reading one stream of text and guessing the next word — mathematically, it can run two trains of thought and return the average of where each train would have gone. That's surprising given how aggressively non-linear the model's insides are, and it's useful: you can get two answers for one pass, and the ability can be turned back on with a bit of fine-tuning if training wore it down. The same property means "one prompt" isn't necessarily "one instruction" — worth watching.
