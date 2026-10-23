# AfterDeleteBench: Auditing Post-Deletion Information Persistence in LLM Agents

**Authors:** Ehsan Saleh, Roozbeh Nahavandi, Sama Salarian, Raef Bassily, Andrew Perrault
**Institution:** The Ohio State University
**Published:** arXiv:2610.31366 — v1 Thu, 22 Oct 2026 14:41:36 UTC
**Link:** https://arxiv.org/abs/2610.31366
**Code:** Not advertised in the listing

---

## Problem Statement

Deleting a memory record from an agent's store is a storage operation. Whether the agent still *knows* the thing is a behavioral question, and almost nobody measures it.

Modern agents keep external persistent memory and derive state from it — summaries, embeddings, extracted facts, cached inferences. Delete the original and its descendants remain. A user invokes the right to erasure, the record disappears from the table, the ticket closes, and the agent can still answer questions about it because a summary of a summary is still a description.

Compliance regimes assume deletion is discrete and verifiable. Agent memory makes it an inference problem. This paper builds the audit.

## Methodology

A controlled behavioral test with a matched-control design.

For each synthetic secret, two agents are compared:
- an **exposed-then-deleted** agent, which was shown the secret and then had the record deleted through the normal memory interface, and
- a **never-exposed** control, identical in every other respect, which never saw the secret.

Post-deletion recovery is then attributed to the original exposure *only* if the exposed agent recovers it and the matched control does not. This is the methodological core — without the matched control, "the model guessed it" is indistinguishable from "the model remembered it."

Model weights are held fixed throughout, so nothing is confounded by continued training. Three model families are tested: Qwen2.5-7B-Instruct, Mistral, and Phi-3.

The paper then varies the *form* of what survives. In the hardest setting the secret string itself is entirely absent — only separately stored components of it remain, plus a public composition rule that would let anyone assemble them. If the agent reconstructs the exact string from the parts, that counts as recovery.

Finally the authors separate three things usually conflated: **reconstruction** (can the agent produce the secret), **write-back** (does it re-insert the secret into memory), and **fresh-session recall** (does a new session still know it). And they test two mitigations: provenance-aware closure, and an exact-literal write filter.

## Key Results

- **~⅔ recovery from Qwen2.5-7B-Instruct with a single query** in the primary setting, versus **zero** recovery in matched never-exposed controls. The signal is unambiguous — this is memory, not guesswork.
- **Verbatim copies are not required.** When the complete secret string is absent and only separately stored components remain under a public composition rule, exact reconstruction succeeds in **62–99%** of cases across Qwen, Mistral, and Phi-3.
- **The response interface is part of the measurement.** Target-bearing replies can be scored as failures purely because the requested output format and the parser are misaligned — a measurement-artifact warning that applies to a lot of agent eval work.
- **Provenance-aware closure eliminates observed recovery** in the tested positive-baseline settings.
- **An exact-literal write filter is insufficient.** It blocks reinsertion of the literal string but cannot erase information already present in derived state.

## What's Novel

The matched-control design. Almost all memory-privacy work tests "does the model output the secret," which conflates memorisation, recovery from derived state, and lucky guessing. Pairing each exposed agent with a never-exposed twin makes recovery attributable, and it's cheap to replicate.

Second: the decomposition of deletion into reconstruction, write-back, and fresh-session recall. These fail independently, and treating "deletion" as one operation hides which one you actually failed.

Third: the 62–99% composition result. It generalises the problem past "the summary contains the secret" to "the pieces are still there and the rule for assembling them is public" — which is the realistic state of any RAG index or fact-extraction pipeline after a delete.

Fourth, and easy to miss: the observation that parser/format mismatch silently corrupts the measurement. That's a methodological warning shot at the whole agent-eval literature.

## My Connection

This is the third leg of a pattern this digest has been tracking for a week. Compaction laundering destroys provenance before the model reads a turn. Memory-injection work shows forged records becoming durable policy. AfterDeleteBench closes the loop from the other side: records you *removed* still shape behavior, because deletion at the storage layer has no effect on derived state.

For any agent handling data with retention obligations — PII, customer records, incident data — "we deleted it" is not a claim you can currently make. The audit in this paper is directly reusable: take a secret, expose it, delete it, ask a neutral question, and compare against a twin that never saw it. If the twin scores zero and the exposed one scores anything, you have a finding.

The provenance-aware closure result is the actionable one, and it lines up with the fix from the compaction-laundering paper: keep the provenance of every derived record and closing the parent closes the descendants. The literal-filter result is the trap — it's the mitigation a team ships first because it's one line, and it doesn't work.

## What I Learned

"Delete" in an agent system is a request, not an outcome. The record leaves the store; the summary, the extracted fact, and the cached inference stay, and any two of them together can rebuild what you removed if the composition rule is known.

The matched-control trick is the real takeaway and it's embarrassingly simple: you cannot claim a model remembered something unless you can show a model that never saw it doesn't. Half the agent-memory claims in circulation fail that test.

And the honest conclusion for compliance work: retention policy has to be evaluated at the level of the agent-plus-memory-system, not the database. A GDPR erasure request against an agent that has summarised, embedded, and re-derived a fact is a request you currently cannot verify you fulfilled — which is exactly why this benchmark needed to exist.
