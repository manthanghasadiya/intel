# Incident-Data Robustness Analysis of the OWASP Top 10 for LLM Applications (2026)

- **Paper:** https://arxiv.org/abs/2608.19266
- **Authors:** Kyriakos "Rock" Lambros, Steve Wilson
- **Institution:** OWASP GenAI Security Project (Top 10 for LLM Applications working group) — independent/exploratory analysis, NOT an official OWASP release
- **Published:** Announced Aug 21, 2026 (v1 submitted Aug 18, 2026)
- **Code/Data:** linked from paper comments

## Problem Statement (plain English)

The OWASP Top 10 for LLM Applications is a community-expert ranking of the risks practitioners *believe* matter most for LLM apps. The authors ask a sharper question: does that expert ranking actually agree with what *real incidents* show? If the expert consensus and the incident record disagree, teams may be defending against the wrong threats.

## Methodology (technical)

- Assembled a large-scale LLM-security incident corpus: **7,714 snapshotted incidents**, **6,639 labeled** against the 20-entry taxonomy, drawn from CVE, GHSA, OSV, and AIAAIC.
- Derived an incident-based risk ranking using a **Bayesian measurement-error model** that corrects each category's incident count for classifier precision and recall (i.e., accounts for mislabeled incidents).
- Compared against the OWASP 2026 candidate list, which blends signals at fixed weights — **0.75 expert vote, 0.25 incident data**.
- Agreement measured with Cohen's κ; robustness checked with a **pre-registered bake-off of four frontier classifiers** and a ground-truth ordering check (Spearman ρ).

## Key Results (with numbers)

- **Weak expert-vs-data agreement: Cohen's κ ≈ 0.20**, 90% interval crossing zero — statistically indistinguishable from no agreement.
- Expert ranking nonetheless **robust**: in the pre-registered bake-off, **no frontier classifier beat the incidence floor's balanced accuracy of 0.863**.
- Ground-truth check left the incidence floor's ordering intact: **Spearman ρ = 0.918** against held-out truth.
- The 0.75/0.25 blend lets incident data correct consensus without overturning it.

## What's Novel

First large-scale attempt to validate the OWASP LLM Top 10 expert ranking against a real incident corpus, with explicit measurement-error correction and a pre-registered classifier bake-off. It quantifies (and limits) how much incident data should weight against expert judgment.

## My Connection (to Manny's work)

Manny runs AI security operations where prioritization is everything. This paper gives a defensible method for grounding risk rankings in incident data rather than vibes — and a caution: expert consensus (κ≈0.20) and incident reality barely correlate, so any LLM-security scoring framework should be periodically re-validated against a labeled incident corpus the way this paper does.

## What I Learned (plain English)

- Expert-ranked threat lists and actual incident data can disagree almost completely — even a "robust" expert list needs data-backed calibration.
- Incident counts are noisy; Bayesian correction for classifier precision/recall is a cheap way to de-bias them.
- A pre-registered bake-off is a strong pattern for settling "which classifier/ranking is best" debates without cherry-picking.
