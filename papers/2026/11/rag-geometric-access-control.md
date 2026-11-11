# Isolation Is Not Confidentiality: Security Boundaries of Geometric Access Control in RAG

**Authors:** Steve Nouyep, Maxime Puys, Sébastien Salva
**Published:** 2026-11-10 (arXiv v1)
**Paper:** https://arxiv.org/abs/2611.14061
**Code:** Not stated in the metadata
**Institution:** Not stated in the arXiv metadata (authors affiliated with French university labs)

---

## Problem Statement

RAG systems put documents from *different* access-control policies into *one* vector database. The naive fix — separate collections per policy — destroys the thing that made vector search useful: you can no longer do a single similarity search across everything.

The recent research fix is elegant: **search-preserving geometric access control.** Transform each embedding so that documents under different policies live in different regions of the embedding space, while cosine similarity within a policy is preserved. One search, correct results, nothing crosses a policy boundary. Papers in this line claim retrieval works *and* unauthorized content is unreachable.

This paper asks the question those papers didn't: **isolation is one property, confidentiality is several.** Search-preserving geometry may prevent cross-policy *retrieval* while leaking, forging, or revealing policy membership in other ways.

## Methodology

A **unified security analysis** of the geometric-transform paradigm, splitting the vague claim "secure RAG" into four independently checkable objectives:

1. **Policy isolation** — can a query under policy A retrieve documents under policy B?
2. **Representation confidentiality** — given the protected representations stored in the vector DB, can anyone recover the original embeddings?
3. **Query forgery resistance** — can an attacker construct a query that is *valid* for a policy they do not hold?
4. **Policy-membership hiding** — can an observer tell which policy a given document or query belongs to?

Two adversary models:

- **Curious cloud provider** — sees the protected representations at rest in the vector database. (The standard "honest-but-curious vector DB" threat model.)
- **Curious user** — holds **aligned pairs** of original embeddings and their protected counterparts, e.g. because they can embed text they authored and observe where it lands. This is the attacker nobody models, and it is trivially realizable by any user with write access to the corpus.

Evaluation: the **available implementations** of the published geometric-transform mechanisms, across **three encoders** (to check whether the results are encoder-specific).

## Key Results

- **Retrieval utility: preserved.** Retrieval still works. This is not a "the method is broken" paper in the functional sense.
- **Policy isolation: strong.** Cross-policy retrieval is genuinely prevented. The claim the original papers make is true.
- **Representation confidentiality: FAILS.** A curious user holding a few hundred aligned pairs **recovers the original embeddings**. The geometric transform is a fixed, invertible-ish map; aligned pairs are enough to solve for it.
- **Query forgery resistance: FAILS.** The same user can **forge policy-compatible queries**. Cost rises with encoder dimension for per-policy *secret* transformations — meaning the security margin is a function of embedding size, not of a key.
- **Policy-membership hiding: FAILS.** Policy-dependent geometry **leaks policy membership** even when unauthorized retrieval is prevented. An observer learns *which policy bucket* a vector belongs to from its position, even without reading its content.

Conclusion: the four properties are **distinct security objectives**, and satisfying one tells you nothing about the other three. Any system claiming "RAG access control solved" must state which of the four it actually delivers.

## What's Novel

- **The decomposition itself.** RAG access-control literature treats "isolation" as the goal. This paper shows isolation is the *easiest* of four properties and the least informative.
- **The aligned-pairs adversary.** Everyone analyzes the curious vector-DB operator. The curious *user with write access* — who can generate embedding pairs at will — breaks confidentiality and forgery resistance with a few hundred samples, and this attacker is inside the trust boundary of most RAG deployments.
- **Membership leakage from geometry.** A side channel in the *shape* of the embedding space rather than in the returned results. Even a perfectly enforced access check leaks the policy label.
- **Encoder-dependence quantified:** confidentiality and forgery costs scale with encoder dimension, which means "pick a bigger embedding model" is accidentally a (weak) security control — and a fragile one.

## My Connection

RAG access control is agent-context security, and this is the paper that tells you where the real boundary is:

- **For agent design:** if an agent retrieves from a multi-tenant index, "the retriever can't return other tenants' chunks" is not a confidentiality guarantee. Embedding inversion, query forgery, and membership leakage all survive that check. The defensible architecture is to keep protected representations out of the index entirely — per-tenant keys with actual encryption, or physically separate stores where the isolation claim needs to be strong.
- **For red teaming:** the aligned-pairs attack is a cheap, high-yield test you can run against any geometric-access-control RAG product. Write text you control, embed it, observe where it lands, repeat a few hundred times, solve for the transform. That is an afternoon, and it produces a confidentiality break with a reproducible artifact.
- **Connects to earlier work in this stream:** the same "isolation ≠ confidentiality" logic shows up in the SGLang/NEAR AI incident today — a TEE isolates the host and says nothing about the software inside. Two different layers, identical reasoning error.
- **Caveat:** the evaluation covers available implementations of a young research line, not a production system. Treat it as a class-level warning about the paradigm, not a CVE against a vendor.

## What I Learned

Hiding documents in a different corner of the embedding space is not the same as keeping them secret. If someone can see where their own text lands, they can work backwards and figure out the transformation — a few hundred tries and the "protections" come apart, including the ability to craft a query that the system will happily treat as authorized. And even when the wall holds, the wall's *shape* tells an observer which room a document is in. Four different questions were being answered with one word — "isolated" — and the only one that word actually answers is whether a search returns the wrong document. Everything else, the content, the queries, the labels, has to be protected by something stronger than geometry: keys, real encryption, or separate stores.
