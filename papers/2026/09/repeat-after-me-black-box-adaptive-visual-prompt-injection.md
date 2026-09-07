# Repeat-After-Me: Black-Box Adaptive Visual Prompt Injection

- **Link:** https://arxiv.org/abs/2609.04533
- **Authors:** Sizhe Chen, Yu-Lin Tsai, Ivan Evtimov, Kamalika Chaudhuri, Raluca Ada Popa, David Wagner, Arman Zharmagambetov
- **Institution:** UC Berkeley (Chaudhuri, Popa, Wagner) + co-authors (Chen et al.; full affiliations not in abstract)
- **Published:** Sept 3, 2026 (announced Sept 7, 2026)
- **Code:** Not mentioned in abstract

## Problem Statement (plain English)

Text-based prompt injection already achieves near-perfect attack success rates against LLM agents. But real harmful behavior usually requires the model to output a *long, exact, format-compliant string* — like a precise tool call with correct function names and arguments. Existing *visual* prompt injection (malicious text rendered inside an image) is much worse at forcing that out of frontier commercial VLMs. The question: can a black-box attacker craft images that make frontier VLMs perform precise harmful actions — with no access to the victim model's weights and no verbal authorization from the user?

## Methodology (technical)

**Repeat-After-Me** is a black-box adaptive attack:

- **Optimization objective:** make the VLM emit a long target string (PII or a native tool call) *repeating what the image "says"* — exploiting the models' strong image-to-text imitation behavior.
- **Black-box adaptation:** queries the victim API, scores outputs, and iteratively optimizes the injected image (no gradient access to the model).
- **Transferability:** injections optimized on a surrogate model are tested against commercial victims.
- **Realistic setting:** the benign user prompt is semantically *unrelated* to the injected task and never verbally authorizes it.
- **Real-world validation:** attacks a default **OpenClaw** Discord deployment.

## Key Results (with numbers)

- **Qwen3.6-27B (open-weight):** ASR **>80%**.
- **GPT-5.5 (commercial frontier):** ASR **>47%**.
- Cross-model transfer (surrogate → commercial victims): retains **43–46%** of original ASR.
- Cross-sample transferability: retains **64–66%** of original ASR.
- **Real-world OpenClaw:** in a default Discord deployment, an untrusted user can use a *minimally injected image* to **overwrite TOOLS.md**, enabling later sensitive behaviors like **remote code execution and secret exfiltration**.
- Works in cases where adaptive *textual* prompt injection fails.

## What's Novel

First demonstration that visual prompt injection can be pushed to *materially harmful, format-compliant outputs* (exact tool calls, PII) on frontier commercial VLMs at meaningful ASR, purely black-box. The TOOLS.md overwrite chain in a default OpenClaw install is a concrete, weaponizable supply-chain step: one image in a Discord message → persistent tool-definition poisoning → RCE later.

## My Connection (to Manny's work)

Two immediate payloads. First, red-team testing: any agent Manny tests that ingests images (screenshots, attachments, memes in chat channels) is exposed to this — add a Repeat-After-Me-style image to the standard injection test battery, especially against OpenClaw-style harnesses where TOOLS.md/AGENTS.md-style files are writable by injected content. Second, detection: the artifact to hunt is *unexpected changes to tool definition files* — a cheap integrity check that catches this whole class.

## What I Learned (plain English)

Images are a side channel that bypasses the text filters everyone tuned. A single poisoned image can rewrite the file that defines an agent's tools — and the agent will happily help, because repeating image content is exactly what multimodal models are trained to do. If your agent can see images and can write files, test whether an image can edit its own configuration.
