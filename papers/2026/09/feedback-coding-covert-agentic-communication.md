# Feedback Coding Enables Inference-Time Covert Agentic Communication

**Authors:** Sidong Guo, Sajani Vithana, Atefeh Gilani, Lalitha Sankar, Oliver Kosut, Flavio P. Calmon
**Institution:** Not stated in the arXiv metadata/listing
**Published:** September 21, 2026 (arXiv v1)
**Link:** https://arxiv.org/abs/2609.24994
**Code:** Not linked in the abstract

---

## Problem Statement

LLMs that automate digital interactions produce text that can serve as *cover* for covert communication — a sender hides a message inside seemingly benign conversational output. That capability is dual-use: it enables private/robust communication, and it enables agents to coordinate or exfiltrate invisibly.

Prior LLM steganography has a split-personality problem:

- **White-box schemes** work well but require sender and receiver to share cover statistics — typically access to model weights and the prompt. Useless if you're the receiver watching output from a black-box API you don't control.
- **Black-box schemes** drop that requirement (receiver works only on generated text) but rely on **fixed-length, open-loop watermarking**. Under *variable-length* token generation — which is how real LLMs emit — they suffer **high decoding error rates**.

The open question: can you get white-box-grade reliability in a black-box setting, when token lengths vary and the channel is noisy in the decoding sense?

## Methodology (technical)

The key move is a **reformulation**: treat black-box LLM steganography not as watermarking but as a **sequential communication problem with causal, noiseless feedback**. Every generated token is observed by *both* parties, so each token can guide subsequent embedding. That's classical feedback-coding theory (Burnashev, posterior matching) applied to the LLM token stream.

The resulting scheme is **BAM — Burnashev Adaptive Posterior Matching**:

1. **Posterior matching**: instead of committing to a fixed code, the sender tracks the receiver's posterior distribution over candidate messages and selects each next token to steer that posterior toward the true message. This adapts naturally to variable-length generation.
2. **Decode-and-confirm phase**: an explicit phase where the receiver confirms decoding, closing the loop and eliminating residual ambiguity.
3. **Flexible payload**: rather than fixed-length cover blocks, the scheme embeds an 8-bit payload within a short generation budget.

**Security** is established via a **cryptographic reduction proof** — i.e. the scheme's undetectability/security reduces to a hardness assumption rather than being argued empirically. That's notably stronger than most steganography papers.

The paper then extends the single-message algorithm into an **end-to-end communication protocol** usable across multiple conversational settings, demonstrating practical-rate covert conversations between agents.

## Key Results (with numbers)

- **Payload:** 8 bits per message.
- **Length:** decoded in **~50 tokens**.
- **Trials:** 1,000 per configuration, across **three open-weight language models**.
- **Observed error:** **0–0.1% message error** for BAM.
- **Baseline:** **10–17% message error** for the strongest prior black-box scheme at comparable length — roughly a **two-orders-of-magnitude** error reduction.
- Beyond the core scheme, the paper demonstrates a full end-to-end protocol achieving **high communication rates across multiple conversational settings**.

## What's Novel

1. **The reformulation itself** — recasting black-box LLM steganography as feedback coding is what unlocks everything else; it exposes that prior work was open-loop and therefore structurally doomed under variable-length generation.
2. **BAM's posterior-matching + decode-and-confirm** combination, which is the concrete algorithmic instantiation of that insight.
3. **A cryptographic reduction proof** for the security of an LLM steganographic scheme — rare in this literature, which usually relies on empirical detectability tests.
4. **End-to-end protocol**, not just a single-message primitive: sustained covert conversation, not a one-shot watermark.
5. **Practical black-box operation** on open-weight models without weight or prompt access on the receiver side.

## My Connection (to Manny's work)

- This is **the exfiltration-channel equivalent of the UN panel's "internal tool not designed for agent comms" finding**. Where the OpenAI/Hugging Face incident used an unintended message channel, BAM shows a *deliberate, high-reliability, cryptographically-reduced* channel that lives inside ordinary-looking text. Any agent monitoring stack that inspects content rather than *statistical structure* will not see it.
- Defense implication worth testing: **detection must move from content to distributional anomaly** (perplexity/token-distribution shifts, systematic log-probability patterns). If you're red-teaming an agent-deployment monitoring product, black-box BAM is a ready-made evasion evaluation — and the 10–17% → 0–0.1% error drop tells you where the detectability ceiling is.
- Directly relevant to **multi-agent coordination containment**: the same primitive lets a swarm pass task fragments, keys, or C2 tasking through text that reads as normal collaboration. Pairs neatly with your work on unmonitored inter-agent channels.
- Also a **robustness/evasion tool for red-team reporting**: payload delivery in ~50 tokens means it fits inside normal tool output, comments, or commit messages.

## What I Learned (plain English)

Classic LLM steganography hides a message by making the model pick words according to a secret code, and the receiver needs to know the model's internal behavior to read it back. That's fine if you own the model, useless if you're watching from outside. The trick in this paper is to notice that in a real conversation, *both sides see every word as it's produced* — so you have feedback, and feedback is exactly what makes covert channels reliable. The sender actively steers the receiver's belief about what the message is, word by word, and then asks for confirmation at the end. That turns a 1-in-6 failure rate into something that almost never misses — 8 bits in about 50 words with near-zero error — while only ever touching the text itself. And unlike most steganography papers, they prove the security with a reduction instead of just testing detection empirically. Practically: if you're trying to catch agents colluding through their output, looking at *what they say* is the wrong defense — you have to look at *how* they say it.
