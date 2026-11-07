# Escalate or Close: How Fast-Tier Models Handle Held-Out Web Attack Families in Two-Tier Triage

**Authors:** Maciej Kisielewicz, Michał Kędziora, Ngoc Thanh Nguyen
**Published:** Submitted November 5, 2026 (arXiv:2611.08229)
**Link:** https://arxiv.org/abs/2611.08229
**Length:** 20 pages, 4 figures, 24 tables

---

## Problem Statement

Two-tier triage is the standard architecture for LLM-assisted security operations: a cheap, fast model decides most web requests and escalates only the remainder to a slow, expensive tier. The structural flaw is that **the fast tier's decision to close a request is final** — an attack the fast tier closes is never examined by anyone.

The paper asks the question that benchmark-leaderboard numbers hide: how does a fast tier behave on attack families it was **never trained on**? Novel attack classes are exactly what matter in production, and exactly what a supervised detector has no signal for.

## Methodology

The experimental design is the contribution. The authors hold each of **seven attack families** completely out of training, recalibration, and threshold selection. They then splice the held-out payloads into parameters of **real browsing traffic**, so request shape is realistic rather than synthetic.

Five classes of comparator are run:

1. **Zero-shot decision models** — models that return a probability for a typed question (Jev and Clef are the hosted ones named).
2. **Supervised detectors** (including ModernBERT).
3. **A local language model.**
4. **The OWASP Core Rule Set** — the deterministic baseline.
5. **An anomaly baseline.**

Two controls deserve attention: the **5% benign budget** (thresholds are set so false positives stay under 5%), and **benign twins** — exploit requests with the attack replaced by legitimate values, used to check whether a model is detecting the attack or merely the shape of the request.

## Key Results

**Supervised detectors generalize badly to held-out families.**
- ModernBERT, over 10 training runs, gives **37% to 60% of held-out path traversal a probability below 0.05** — i.e. a confident benign verdict on the majority of a novel attack class.
- Within a 5% benign budget it escalates only **50% to 81%** of held-out traversal, versus nearly all of the same requests when traversal is in the training set.

**Hosted decision models look much better — with a caveat.**
- Jev and Clef escalate **at least 97.6% of every family** within the same budget, and their budget decisions do not depend on the held-out family.
- Caveat: their training data is undisclosed and may contain these public attack families, so the held-out design cannot fully verify the result.

**But the shape control exposes them.**
- **Jev escalates 37% of benign twins** — it is partly detecting request shape rather than attack content.
- Detectors trained on older exploits **learn what scanner requests look like**, which is a quiet form of overfitting to tooling.

**Adaptive attack works.**
- A search guided by Jev's probability gets **43% of held-out attacks past Jev within 300 queries**.
- Jev's answers are **non-deterministic**: asked again, it escalates those same requests in **60% of its answers**.

## What's Novel

1. **True held-out families, not held-out samples.** Most triage evaluations split by sample within known classes, which measures memorization. Holding out whole families measures whether the tier can recognize an unseen attack *concept*.
2. **Splicing into real browsing traffic.** Keeps payload realism and request-shape realism simultaneously — the two are usually traded off.
3. **Benign twins as a shape control.** This is the methodological star: it separates "detects the attack" from "detects a scanner." Any eval of an ML security gate should copy it.
4. **Quantifying non-determinism as an attack surface.** "60% of the time it escalates" is not a rounding error, it is a queryable oracle.

## My Connection

- **Every "AI SOC" claim is now falsifiable.** If a vendor says their fast tier catches novel attacks, ask for held-out-family escalation rate within a benign budget, and ask for the benign-twin rate. Without the twin control, their number is probably measuring request shape.
- **Direct offensive technique.** Probability-guided search at 300 queries to get 43% past a hosted decision model is a practical, cheap evasion loop — a red-team primitive for any target that gates traffic with a model. Combine with the non-determinism finding: retry the same payload, since the model's answer flips.
- **Detection engineering.** The 50–81% escalation on held-out traversal means the slow tier must be assumed to never see half of a novel class. Budget slow-tier capacity accordingly, and do not treat "closed by fast tier" as "benign."

## What I Learned

A classifier's accuracy on the classes you trained it on tells you almost nothing about the classes that will actually hurt you. The deeper lesson is about the *control* the paper invented: benign twins. When a detector looks good, the first question is never "how accurate is it" but "what else correlates with the positive class, and is the model reading that instead?" Jev escalating 37% of benign twins means a third of its "detections" are the shape of the tool, not the attack — and in a tiered system, that is not a false-positive annoyance, it is an evasion channel, because the attacker controls the shape.
