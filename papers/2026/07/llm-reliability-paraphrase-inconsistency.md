# Paper Breakdown: Same Question, Different Answers — Evaluating LLM Reliability Beyond Accuracy

- **Link:** https://arxiv.org/abs/2607.22554
- **Authors:** Kazem Faghih, Yize Cheng, Shoumik Saha, Mobina Pournemat, Armin Gerami, Soheil Feizi
- **Published:** July 28, 2026 (Tuesday)
- **Code:** Not specified

---

## Problem Statement (Plain English)

LLMs often score high on accuracy benchmarks, but these benchmarks test each question once. In the real world, the same question can be phrased dozens of different ways. This paper asks: if you ask the same question in multiple different ways (paraphrases that preserve meaning), does the model give the same answer? The finding: often no. A model might get a question right when phrased one way but wrong when phrased another — even though both phrasings mean the same thing. This means accuracy metrics are misleadingly optimistic.

## Methodology (Technical)

- **Datasets:** 4 benchmarks spanning factual QA (MMLU, TriviaQA) and mathematical reasoning (GSM8K, MATH)
- **Models:** 13 models across different families and sizes (including GPT variants, Llama, Claude, etc.)
- **Approach:** For each question, they generate multiple meaning-preserving paraphrases using a separate LLM, then measure:
  1. **Accuracy** per paraphrase (expected to be similar)
  2. **Answer flip rate** — how often the same question gets a different answer across paraphrases (the key metric)
  3. **Latent knowledge recovery** — whether the correct answer appears in *any* paraphrase for a given question
- They also test a **self-paraphrasing strategy**: ask the model to rephrase its own question, then answer, then aggregate.

## Key Results (with Numbers)

- **Instance-level mismatch rate:** >23% for many model-question pairs — the model gives contradictory answers to the same question rephrased
- **Conditional flip rate:** When looking only at questions answered correctly in their original form, flip rates on alternative phrasings were even higher
- **Latent knowledge exists:** For most questions, at least one paraphrase produces the correct answer — meaning the model "knows" the answer but doesn't consistently access it
- **Self-paraphrasing helps:** Asking the model to rephrase its own question and then answer recovers some of this latent knowledge, improving aggregate accuracy
- Overall accuracy across paraphrases changes modestly, but the paper argues that **average accuracy is a poor metric** — what matters is consistency

## What's Novel

- Moves beyond average accuracy to **instance-level reliability** as the primary metric
- Demonstrates that "single-prompt correctness" is a poor indicator of actual knowledge
- The **self-paraphrasing** technique is a practical, inference-time intervention that requires no retraining
- Tests across both factual and reasoning domains — the problem is general, not domain-specific

## My Connection (to Manny's Work)

This is directly exploitable in AI red team operations:
- A single "successful" jailbreak or bypass test is unreliable — you need to test across paraphrases
- Conversely, if you find one prompt that reliably flips the answer, you have a repeatable attack vector
- The self-paraphrasing insight means that asking models to rephrase their own questions could help them bypass guardrails — or could help defenders detect inconsistencies
- For agent security: if an agent is supposed to make a consistent decision (e.g., "is this action safe?"), inconsistency across equivalent inputs is a vulnerability

## What I Learned (Plain English)

Model accuracy numbers are lying to us. A model can score 90% on a benchmark but give a different answer to the exact same question asked slightly differently 23% of the time. The knowledge is in there somewhere (one phrasing usually works), but the model can't consistently retrieve it. This is less like a human who knows the answer and more like a student who passes a test by luck. The practical takeaway: don't trust a single response from an AI — especially for security-critical decisions.
