# CTIFoundry: An Agent-Native Corpus Scaffold for Cyber Threat Intelligence

- **Link:** https://arxiv.org/abs/2608.18613
- **Authors:** Yutong Cheng, Changze Li, Qian Cui, Wei Ding, Lingzhi Wang, Yan Chen, Peng Gao
- **Institution:** Academic multi-institution team (preprint metadata omits affiliations)
- **Published:** 2026-08-19 (v1)
- **Code:** Not provided in preprint metadata

## Problem Statement (plain English)

Threat intelligence is increasingly consumed by LLM agents that compose multi-step investigations at query time — but the *corpus side* hasn't caught up. CTI reports and vulnerability databases are still packaged as opaque chunks behind an embedding index (flat RAG). The authors argue this substrate — not model capability — is the bottleneck on agentic CTI investigation: an agent asked to trace a CVE to its ATT&CK technique to affected products has to guess the links that the corpus already encodes implicitly.

## Methodology (technical)

At **build time**, CTIFoundry materializes the latent structure of a CTI corpus:
- A **deterministic ontology graph** over four authoritative knowledge bases (CVE, CWE, CAPEC, ATT&CK) whose official cross-references become typed, traversable edges.
- A **span-grounded report layer** where canonical, alias-resolved cross-vendor entities index provenance-carrying chunks.
- **Hybrid dense + lexical retrieval surfaces** (not embeddings-only).

At **query time**, this structure is exposed through **seven typed tools** and **three procedural skills**, mounted on a stock open-source agent harness. The evaluation swaps only the action surface — same harness, same models, same benchmark — isolating the corpus contribution.

## Key Results (with numbers)

- On the public **CTIConnect** benchmark, swapping only the action surface lifts the identically-harnessed agent by **+0.19 to +0.28 overall F1** across a four-model, two-provider panel.
- **A small model on CTIFoundry surpasses a flagship model on the flat substrate** — corpus structure compensates for model capability.
- The gain is not bought with search effort: on both Claude models, the scaffolded agent is *more accurate at roughly half the tool calls*.
- Ablation: typed structure carries the larger share of the gain; procedural skills convert structure into discipline; the two compose **super-additively** (skills bind only to structure that exists).

## What's Novel

- Reframes the agentic-CTI problem as a **data-substrate problem**, not a model problem — most prior work optimizes the harness or the model.
- Explicit, typed, *traversable* ontology edges between CVE/CWE/CAPEC/ATT&CK replace implicit similarity search — the agent walks the graph instead of hoping RAG retrieves the right neighbor.
- Provenance-carrying, alias-resolved entity grounding for report chunks (cross-vendor entity canonicalization).
- Rigorous ablation isolating structure vs. skills vs. their composition.

## My Connection (to Manny's work)

For an AI red team operator, this is a force multiplier on both sides: (1) **as a defender/analyst tool**, agentic CTI investigation with typed graph traversal means faster, cheaper, more accurate correlation of IOCs, CVEs, and ATT&CK techniques — the SilkParasite-class campaigns in today's digest are exactly the multi-hop investigations this targets; (2) **as an attack surface**, the "seven typed tools" on a stock harness is a classic MCP-style tool layer — permission boundaries, prompt injection through poisoned report chunks, and tool-abuse paths all need red-teaming before deployment. The lesson that substrate beats model is also a lesson for Manny's own tooling: give agents structure, not text blobs.

## What I Learned (plain English)

- RAG is the wrong default for knowledge-heavy agent tasks: an embedding index is a lossy, implicit encoding of relationships that CTI already expresses explicitly (CVE → CWE → CAPEC → ATT&CK).
- Making structure *typed and traversable* (edges with meaning, tools that walk them) converts retrieval into reasoning — and measurably beats raw model capability: a small model + good substrate > a flagship + flat RAG.
- Fewer tool calls at higher accuracy means structure also reduces the attack surface and the token cost — efficiency and security align.
- Super-additivity (structure + skills > sum of parts) is a design principle: tooling should bind skills to existing structure rather than bolt on generic instructions.
