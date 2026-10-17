# Poisoning the Neighborhood: A Topology-Aware Topic-Level Attack on Large-Scale HNSW-based RAG Systems

**Authors:** Yufei Chen, Yao Wang, Haibin Zhang
**Published:** 2026-10-15
**Link:** https://arxiv.org/abs/2610.21464
**Subjects:** cs.CR
**Code:** Not linked in the abstract.

---

## Problem Statement

RAG systems ingest documents from public and third-party sources, which makes them poisonable. But nearly all published poisoning attacks assume **exact retrieval**: the poisoned document is scored against the query and competes in a global ranking, so an attacker just needs high similarity to the target query.

Production does not work that way. At scale, retrieval runs on approximate nearest-neighbour indexes — overwhelmingly HNSW (Hierarchical Navigable Small World). HNSW search has a small budget (`ef_search`), bounded graph traversal, and heavy **local competition** from benign documents in the same neighbourhood. Under those constraints, a poisoned document with high global similarity may simply never be reached. The result: attacks that look strong in papers degrade badly in deployment, and defenders get a false sense of coverage.

The paper asks: **can poisoning be designed for the graph, instead of for the ranking?**

## Methodology

**1. Attack structure — Hub-Satellite.**
Poisoned documents are organized into two roles:

- **Satellites** — placed across semantic regions to provide competitive entry points, so the search lands near poison regardless of which region the query falls in.
- **Hubs** — strengthen local cohesion inside each neighbourhood and create cascading retrieval, so once the search enters a poisoned region it keeps pulling poison into the result set.

This exploits a property of HNSW specifically: the graph builds links from local proximity, so clusters of mutually-similar poison get densely connected whether or not the attacker intended it.

**2. Generalization to unseen queries — topic-level, not query-level.**
Instead of targeting specific queries, the attack targets a **topic**, so it fires on queries the attacker never saw. This is the practical difference between a demo and a campaign.

**3. Generation-stage payload — entity-claim promotion.**
Retrieval alone is not enough; the retrieved text must change the answer. The attack combines **evidence anchoring** (the poison reads as sourced, quotable evidence) with **entity-claim coupling** (the target entity is bound to the target claim in the same span), so the model promotes the entity via the claim across different query regions.

**4. Evaluation at deployment scale.**
2M-document Wikipedia corpus, plus domain-specific experiments. Metrics: retrieval success (does poison make the top-k) and attack success (does the generated answer carry the claim).

**5. Defense evaluation.**
Existing poisoning defenses are run against the attack.

## Key Results

| Metric | Result |
|---|---|
| Retrieval success (2M-doc Wikipedia) | > 72% |
| Attack success (claim in generated answer) | 31.29% |
| Performance vs. baselines | Substantially better |
| Existing poisoning defenses | Cannot mitigate |
| Hub-Satellite topology formation in HNSW | Confirmed by structural analysis |

The structural analysis is the interesting part: the authors show the Hub-Satellite topology **emerges in the HNSW graph itself**, which is the causal explanation for why retrieval works. It is not a similarity trick — it is graph engineering.

## What's Novel

- First poisoning attack designed against **HNSW's search mechanics** (budget, reachability, local competition) rather than against global similarity ranking.
- **Topic-level generalization** — survives unseen queries, which is what makes a poisoning attack operationally meaningful.
- **Hub-Satellite topology** as an explicit adversarial structure, with empirical confirmation that it forms in the index.
- **Entity-claim promotion** as a generation-stage companion to retrieval-stage poisoning — most papers stop at "poison was retrieved."
- Shows the standard defense set does not transfer to the graph setting.

## My Connection

Manny's offensive work on agent infrastructure runs straight through RAG:

1. **Attack path for agent knowledge bases.** Any agent with an ingest pipeline (docs, tickets, scraped pages, customer-uploaded files) is a candidate. The operational recipe is: characterize the index (is it HNSW? what `ef_search`?), then place satellites across topic regions rather than one high-similarity document.
2. **Defense testing.** "We have a poisoning detector" is not a control if the detector was validated under exact retrieval. This paper gives the exact test harness to break that claim: 2M-document scale, topic-level queries, measured attack success at generation.
3. **Red-team reporting angle.** The 72% retrieval / 31% attack-success split is a useful calibration for writing findings — retrieval success is not impact; the claim has to survive into the answer.

For defensive tooling, the implication is that retrieval-time anomaly detection should look at **graph structure** (dense isolated clusters, unusual in-degree within a neighbourhood), not just text similarity to known-poison patterns.

## What I Learned

An approximate index is not a weaker version of an exact one — it is a different attack surface with its own physics. In HNSW, proximity is a graph property, so an attacker who can place a cluster controls reachability. And a poisoning attack that only wins on queries the attacker chose is a lab result; winning on a topic is a campaign.
