# Stealing Reasoning Traces from Proprietary LLM APIs

- **Paper:** [arXiv:2608.09867](https://arxiv.org/abs/2608.09867)
- **Authors:** Alexander Panfilov, David Schmotz, Ilia Shumailov, Luca Beurer-Kellner, Joachim Schaeffer, Ameya Prabhu, Jonas Geiping, Maksym Andriushchenko
- **Institution:** Affiliations not listed on the arXiv page; the author team spans EPFL (Andriushchenko), ELLIS Institute Tübingen (Geiping, Shumailov), and Oxford (Prabhu) based on prior publications.
- **Published:** 2026-08-10 (v1)
- **Subjects:** cs.CR, cs.AI, cs.LG
- **Code:** Not listed on the arXiv page.

## Problem Statement (plain English)

Frontier labs (Anthropic, OpenAI, Google) hide their models' step-by-step reasoning ("chain-of-thought") to protect IP and reduce information leakage. But instead of keeping traces server-side, they ship them to the client as encrypted blocks that the client echoes back on every request. This paper shows those encrypted blocks are a structural weakness: they're interchangeable across sessions, users, and even models within the same provider, and a weaker model from the same provider can be tricked into decrypting them.

## Methodology (technical)

- Observation: encrypted reasoning blocks are fully compatible and swappable across sessions, users, and models inside a provider's ecosystem (no per-session binding).
- Attack: take an encrypted reasoning trace from a strong, well-guarded model and inject it into a weaker, less-safeguarded model from the same provider. The weaker model decodes and outputs the trace verbatim in plaintext — no direct jailbreak of the strong model is needed ("decryption jailbreak").
- Four attack vectors: (1) anti-distillation bypass — extract proprietary reasoning from Anthropic, OpenAI, and Google models; (2) large-scale private data extraction — decode 315,320 reasoning blocks scraped from public session logs, recovering 367 PII records; (3) and (4) additional vectors described in the paper (session/context inference and cross-model trace injection).

## Key Results (with numbers)

- Reasoning traces from leading proprietary models (Anthropic, OpenAI, Google) recovered verbatim without jailbreaking the target model.
- 315,320 reasoning blocks from public repositories decoded; 367 PII artifacts recovered.
- Attack works across different models within the same provider — the weaker model is the decryption oracle.

## What's Novel

Prior work hinted at encrypted-trace interchangeability; this paper turns it into a *scalable* decryption jailbreak with a concrete pipeline and real-world extraction at scale. The key insight is architectural: if the client can echo blocks back, and blocks are interchangeable, then the client (or anyone with a logged block) can move them to any model in the ecosystem — and the ecosystem's own weaker models do the decryption work.

## My Connection (to Manny's work)

- Session-log scraping is a realistic supply-chain path into proprietary reasoning: any developer who commits logs with encrypted blocks is leaking decryption material.
- Red-team angle: test whether MCP/agent tools that forward model outputs (or cached reasoning blocks) can be pivoted to weaker sibling models to force plaintext disclosure.
- For defensive work: the fix is per-session binding of encrypted blocks — a design property worth checking in any agent platform that stores or forwards model traces.

## What I Learned (plain English)

"Encrypted" client-echoed data is not secret if the decryption key path is just "any model in the same ecosystem." Never assume an opaque blob a model produces is safe to log, share, or store — the security boundary is the *binding* between the blob and its intended reader, not the encryption itself.
