# Micro-Collaborative Poisoning: A Distributed Attack on RAG Systems

**Authors:** Pedro Pereira, Eva Maia, Isabel Praça
**Institution:** ISEP / GECAD (Polytechnic of Porto) — Portugal
**Date:** Submitted Sep 18, 2026 · cs.CR, cs.AI
**Venue:** 31st European Symposium on Research in Computer Security (ESORICS) 2026 — 2nd Workshop on the Use of Large Language Models for Cybersecurity
**Link:** https://arxiv.org/abs/2609.21573
**Format:** 19 pages, 3 images, 4 tables

---

## Problem Statement (plain English)

Retrieval-Augmented Generation (RAG) grounds an LLM's answers in documents pulled from a knowledge base. That dependency is also an attack surface: if you poison the corpus, you poison the answer. Existing RAG-poisoning attacks tend to concentrate the lie in **one** malicious passage — which makes them detectable, because a single passage that looks nothing like its neighbors stands out to both retrieval heuristics and human review.

This paper asks a sharper question: what if you don't concentrate the lie at all? What if you **split a false target claim across many individually plausible documents**, each of which looks clean on its own, and let the retrieval system assemble the lie for you? That is **Micro-Collaborative Poisoning** — a distributed attack where the harm emerges from the *accumulation* of weak adversarial signals, not from any single dominant poisoned passage.

## Methodology (technical)

- **108 RAG configurations** evaluated by varying six axes:
  1. **Dataset**
  2. **Retriever architecture**
  3. **Retrieval depth** (top-k)
  4. **Database composition** (clean vs. poisoned ratio)
  5. **Number of poisoned databases**
  6. **Generator model**
- The false target claim is fragmented and distributed so that each fragment lives in a locally plausible document; no single document contains the full malicious assertion.
- Measurements focus on how the weak signals **combine in the retrieved context** and influence the downstream generated answer.
- A **document-level poisoning visibility analysis** compares the explicit poisoning signature of micro-collaborative poisoning against direct (concentrated) poisoning.

## Key Results (with numbers)

- Micro-Collaborative Poisoning is **not driven by a single dominant poisoned passage** — the effect is produced by the **accumulation of weak adversarial signals across retrieved sources**.
- **Increasing top-k** and **poisoning multiple databases** both raise the probability that the fragmented signals appear together in the retrieved context (i.e., higher retrieval depth *helps the attacker*).
- **Clean database diversity** and **stronger retrievers** reduce the attack's influence (i.e., corpus hygiene and better retrieval quality are real mitigations).
- **Document-level visibility analysis:** micro-collaborative poisoning achieves **downstream influence while leaving a weaker explicit poisoning signature than direct poisoning** — meaning it is **difficult to expose through isolated document inspection**.
- Tested across 108 configurations (dataset × retriever × depth × DB composition × number of poisoned DBs × generator model), reported in 4 tables / 3 figures.

## What's Novel

- Reframes RAG poisoning from a **point** attack (one bad passage) to a **distributed, emergent** attack (many slightly-off passages), which directly defeats "does any single retrieved document look malicious?" defenses.
- Introduces a **document-level visibility metric** that quantifies the attacker's stealth advantage — the key operational finding: the attack is *harder to see* precisely because it looks clean at the document level.
- Uses a **large factorial sweep (108 configs)** to isolate which system knobs (top-k, DB count, retriever strength, corpus diversity) move the attacker's success, rather than testing a single setup.
- Peer-reviewed (ESORICS 2026 workshop), giving it more weight than the typical preprint on this topic.

## My Connection (to Manny's work)

- **RAG/retrieval poisoning is now a client-facing red-team deliverable.** This paper gives the playbook: don't drop one obviously malicious doc — fragment the claim across plausible documents, turn up retrieval depth, and poison across multiple corpora. It's cheap to demo and hard for a defender to explain.
- **Defensive recommendation:** the results invert a common intuition — "increase top-k for better recall" *widens* the attack surface. For engagements, recommend per-document provenance/consistency checks and cross-document claim reconciliation rather than per-passage scanning.
- **Pairs with the day's other work:** alongside Loopjacking and APort Vault, this completes a triad on "trust boundaries that must be verified at use-time" — context assembly (RAG), action approval (Loopjacking), and payment authorization (OAP).

## What I Learned (plain English)

A lie split into a hundred half-truths is harder to catch than one big lie. When each poisoned document looks unremarkable next to its neighbors, the "does this document look malicious?" check finds nothing — yet the retriever quietly gathers all the fragments into the same context window and the model dutifully assembles the false answer. The uncomfortable part: the system setting most teams tune *up* for quality (top-k) makes the attack *stronger*, because pulling more documents increases the odds that all the pieces show up together. Defense has to reason across documents, not within them.
