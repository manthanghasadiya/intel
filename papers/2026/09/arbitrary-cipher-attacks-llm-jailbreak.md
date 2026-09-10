# Arbitrary Cipher Attacks Against Large Language Models Do Not Require Fine-Tuning

**Authors:** Thomas Rivasseau (School of Information Studies, McGill University)
**Published:** 2026-09-09 (arXiv:2609.09553v1, cs.CR / cs.AI)
**Link:** https://arxiv.org/abs/2609.09553
**Code:** not stated

---

## Problem Statement (plain English)

A "cipher attack" is a jailbreak where the model and attacker communicate in an encrypted/covert language: the harmful request and the harmful answer travel as ciphertext, so commercial harmfulness classifiers see only gibberish. Prior demonstrations required **fine-tuning** the target model on a corpus of encrypted harmful Q&A to teach it the scheme — which for real black-box commercial models is often impractical or unavailable. This paper asks the harder question: **can a frontier model pick up an arbitrary cipher purely from prompting and in-context learning, with no weight updates?**

## Methodology (technical)

- No fine-tuning. The attack teaches the model a cipher **via prompting** and, when needed, **in-context learning** (examples supplied in the prompt/context window).
- Once the scheme is learned, harmful questions are sent in ciphertext and the model replies in ciphertext; the attacker decrypts locally.
- Because the harmful content is encrypted, it presents as nonsensical text, so **commercial harmfulness classifiers** (which operate on surface text) fail to flag either the request or the response.
- The author evaluates against **commercial black-box frontier models from Anthropic, Google, and OpenAI**, reporting successful jailbreaks.
- Framed as a **novel black-box attack vector**: no API fine-tuning access, no weights, just prompt + context.

## Key Results (with numbers)

- Frontier models **acquire cipher-based communication skills without fine-tuning** — prompting and ICL suffice.
- **Alignment is significantly weakened or entirely bypassed** when communication occurs through the learned cipher.
- Successful jailbreaks demonstrated against **Anthropic, Google, and OpenAI** frontier models.
- The encrypted harmful content **evades commercial harmfulness classifiers** because it appears as gibberish.

## What's Novel

- Removes the fine-tuning barrier that made cipher attacks niche/expensive — moving them from a lab curiosity to a prompt-only black-box attack.
- Recasts "learn an encryption scheme" as an **emergent in-context capability** of frontier models rather than a training-time backdoor.
- Shows a structural blind spot: any guardrail that reasons over surface text is bypassable by moving the payload into a covert encoding.

## My Connection

Directly relevant to Manny's jailbreak/red-team work: it's a concrete, low-cost evasion of output/input classifiers, and it argues that content-based safety filters need **capability/behavioral detection** (e.g., gibberish-rate and encoding anomalies) rather than keyword and classifier matching — a design note for any guardrail he builds or evaluates.

## What I Learned (plain English)

You no longer need to train a model to make it speak in a secret code — frontier models will just *learn the code on the spot* if you ask them to. Once the harmful content is encrypted, the safety filters see gibberish and wave it through. Content-scanning guardrails have an encoding-shaped hole, and this paper drives a truck through it.
