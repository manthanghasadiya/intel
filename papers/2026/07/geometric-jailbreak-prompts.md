# Geometric Configurations of Perturbed Jailbreak Prompts

**Link:** [arXiv:2607.20581](https://arxiv.org/abs/2607.20581)  
**Authors:** Lynn Delcon, Andres Algaba, Vincent Ginis  
**Published:** July 24, 2026  
**Conference:** 2nd Workshop on Safe AI (SafeAI)  
**Code:** Not specified

---

## Problem Statement

Perturbation techniques that turn unsuccessful jailbreak prompts into successful ones are evolving rapidly, constituting a major security threat to LLM safety. This paper asks: can we find a geometric signature in the model's internal representations that cleanly separates jailbroken from safe prompts?

## Methodology

The authors investigate internal representations of string-level perturbed jailbreak inputs in small weight models from two families:
- **Qwen-2.5 family:** 1.5B, 3B, 7B instruct models
- **Llama-3.2 family:** 1B, 3B, 3.1-8B instruct models

They explore **two representation spaces:**
1. **Last-layer-last-token embedding space** — the final hidden state of the last token
2. **Top-50 next-token probability space** — the log-probabilities of the top 50 next tokens

They test whether a behavioral hyperplane exists in either space that linearly separates jailbroken (compliant) from safe (refusal) responses.

## Key Results

- **No behavioral hyperplane exists** in either representation space within their refusal-dominated answer set
- The embedding space separates prompts primarily by **spelling and format**, not safety content
- The probability space is effectively **one-dimensional** but more complex to cluster
- Only two token-level correlations with compliance were found across all models:
  - The token "Sure" in Qwen 1.5B showed significant compliance association
  - The tokens "," and "ĊĊ" in Llama 1B showed significant compliance association
- Larger models within each family did not show clearer separation

## What's Novel

- Directly tests the geometric separation hypothesis for jailbreak detection rather than proposing a new attack or defense
- Compares two distinct representation spaces (embedding vs probability) systematically
- Shows that what appears to separate safe from jailbroken prompts is actually *format/spelling*, not content safety

## My Connection

- Directly informs Manny's approach to building jailbreak detectors and guardrails
- Confirms that simple representation-space filtering won't work — any practical defense needs to be more sophisticated
- The token-level correlations ("Sure", ",") are interesting for lightweight heuristic detection in deployed agents

## What I Learned

- Jailbreak detection at the embedding level is fundamentally harder than many assume — there's no clean linear boundary
- The "last token embedding" approach that works so well for classification tasks fails for safety classification
- Small correlations like "Sure" token probability are fragile and model-specific — relying on them would create easy bypasses
- The fact that larger models don't help suggests this is a representational limitation, not a scale issue
