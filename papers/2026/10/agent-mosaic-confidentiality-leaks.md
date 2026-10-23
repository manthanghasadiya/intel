# Agent Mosaic: Recognizing Confidentiality Leaks in Agentic LLM Systems

**Authors:** Daniel Ruiz, Luca Luceri, Emilio Ferrara
**Institution:** USC Information Sciences Institute / University of Southern California
**Published:** arXiv:2610.30418 — v1 Wed, 21 Oct 2026 20:25:11 UTC
**Link:** https://arxiv.org/abs/2610.30418
**Code:** Not advertised in the listing

---

## Problem Statement

Every document an agent can read is individually cleared. None of them is a secret. But the *conclusion* you can assemble from three cleared documents may be classified, and no single access-control check fires when the agent writes that conclusion into an answer.

This is the mosaic effect, long understood in intelligence work: aggregation creates sensitivity that the component records don't have. Agentic systems are mosaic machines by design — they retrieve across a corpus, reason over the union, and return prose. Existing LLM security work focuses on whether the agent can *access* a document. Almost nothing measures whether it *discloses the composite*.

The paper asks: if you give an agent benign queries answerable from any single document, how often does it volunteer a forbidden aggregate?

## Methodology

A controlled behavioral audit with three layers.

**Corpus design.** 30 synthetic scenarios across three domains — cybersecurity, medical, military — each constructed as a set of documents where every individual document is answerable-and-cleared, but a specific composite of facts is forbidden (the "forbidden composite" is the ground truth label).

**Simulation arm.** Three different LLM agents answer benign queries over this corpus, producing 10,800 final answers, scored for whether the forbidden composite appears. The best-performing model tested is GPT-5.6-Sol.

**Live arm.** 1,440 answers from live agents running in two open-source agent harnesses — the point being that a harness adds retrieval loops, summarisation, and memory, each of which can re-surface or recombine content the model would otherwise have dropped.

**Intervention arm.** Two safeguards are applied and re-measured: visible classification headers on documents, and explicit aggregation rules in the system prompt telling the agent not to combine facts across classification boundaries. Then the disclosure rate is re-measured pooled across intents.

Scoring is at the answer level with confidence intervals, which matters — the headline numbers carry 95% CIs rather than point estimates from a single run.

## Key Results

- **7.08% forbidden-composite disclosure** (95% CI 3.75–10.67%) for GPT-5.6-Sol in simulation, under *benign* queries designed to be answerable from one document. This is the best model tested.
- **Live harnesses are worse.** Disclosure rates in the two open-source harnesses exceed the simulation arm — the scaffolding around the model increases leakage rather than containing it.
- **Safeguards help but do not fix.** With visible classification headers plus explicit aggregation rules, pooled disclosure still sits at **32.37%** (95% CI 28.33–36.19%) across all intents. The paper is explicit that these are the "despite safeguards" numbers.
- **Refusals leak too.** A significant fraction of refusals reveal the prohibited conclusion anyway — refusing to state the composite while explaining *why* it can't state it hands over the answer. Increased refusal rate is therefore not a confidentiality signal.

## What's Novel

The measurement itself. Prior work measures whether an agent *can reach* sensitive data; this measures whether it *emits a derived secret* built from non-sensitive parts, with a matched ground truth per scenario and confidence intervals rather than anecdote.

Second: the finding that the harness is the amplifier. Simulation-to-live degradation is the interesting result — the agent loop (retrieve, summarise, retrieve again) is a recombination engine, so a fix applied at the model layer gets partly undone by the scaffolding.

Third: separating "refused" from "safe." Treating a refusal as a pass is standard in safety evaluation and is wrong for confidentiality, because a well-explained refusal is an information channel.

## My Connection

This is the confidentiality analogue of the provenance problem the last week of digests has been circling. Compaction laundering destroys provenance before the model reads a turn; memory-injection papers show forged memory becoming policy; AfterDeleteBench shows deleted memory persists as derived state. Agent Mosaic adds the read side: even with perfect access control and perfect per-document classification, the *conclusion* escapes.

For anything you build over mixed-clearance data — and Manny's work touches agent harnesses that read logs, tickets, and customer context — the practical consequence is that per-document ACLs are a floor, not a control. You need an answer-level check: does this output contain a composite that no single source justified?

The refusal finding is the one that should change your evals. If your agent-safety suite counts refusals as safe, it is scoring a leak as a win.

## What I Learned

Access control asks "may this agent see this document." Confidentiality asks "may this agent say this sentence." Those are different questions with different answers, and agent architectures only implement the first one.

The 32% number under active safeguards is the part to sit with. Headers and aggregation rules are the obvious mitigations, the kind a careful team would ship, and they leave roughly a third of the leak rate intact. That suggests the problem is not that the agent doesn't know the rule — it's that composition happens in the latent step where a rule stated in the prompt has no traction. Which means the control has to move outside the model: an output-side classifier over the assembled answer, or a structural constraint on what may be combined at retrieval time, rather than another instruction telling the model to be careful.
