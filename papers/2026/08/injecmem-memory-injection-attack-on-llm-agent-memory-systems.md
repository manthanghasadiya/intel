# InjecMEM: Memory Injection Attack on LLM Agent Memory Systems

- **Link:** https://arxiv.org/abs/2608.23471
- **Authors:** Hanling Tian, Gengyu Zhang, Zeyang Sha, Jingying Wang, Yuhang Liu, Zhehao Huang, Kun Yang, Xiaolin Huang
- **Institution:** Not stated in abstract (author histories point to Shanghai Jiao Tong University / TU Delft)
- **Published:** 2026-08-24
- **Code:** Not released (framework described as reproducible)

## Problem Statement (plain English)

LLM agents are getting persistent memory subsystems — a store of facts about the user and past interactions that gets retrieved and injected into future prompts. That memory is now a default component, so the question is: can an attacker weaponize it without ever touching the memory store? InjecMEM shows the attack needs only a single interaction with the agent (e.g. one message, one document it reads) to make later, related queries return whatever output the attacker wants.

## Methodology (technical)

The attack targets the retrieval-then-generate pipeline of memory systems with two crafted components:

1. **Retriever-agnostic anchor** — high-recall topical cues engineered so that downstream retrieval consistently associates the injected record with the target topic, regardless of which retriever the memory system uses.
2. **Adversarial command** — a short token sequence optimized to remain effective under uncertain fused contexts, variable insertion positions, and long prompts, so it reliably steers generation once retrieved.

The command is learned via **gradient-based coordinate search**, averaging over synthetic prompt templates and insertion positions during optimization, and extended to **joint optimization across multiple backbone models** to study transferability.

## Key Results (with numbers)

- Achieves reliable **topic-conditioned retrieval and targeted generation** across multiple memory systems and backbone models.
- Attack **remains effective under memory drift** (records changing over time).
- **Non-target queries are unaffected** — the poisoning is selective, making it hard to detect via behavioral sampling.
- Requires only **a single interaction**; no read or edit access to the memory store.
- Transfers across backbones via joint optimization (strength depends on shared representation space).

## What's Novel

- First systematic "memory injection" attack paradigm that treats the persistent memory subsystem as the attack surface, not the prompt.
- Attack is **read/write-free** — the victim's memory store is never touched; the injection rides in through a normal interaction.
- Retriever-agnostic anchors address a real engineering constraint: production memory systems swap retrievers, so attacks tied to one retriever are fragile.

## My Connection (to Manny's work)

Manny builds and audits MCP/agent tooling, where persistent memory is becoming a standard MCP-adjacent service (memories, knowledge-graph servers, vector stores). This paper gives a concrete, testable attack class for that stack: a single poisoned document/tool result can silently steer an agent's future decisions. It also argues for memory-integrity monitoring — probing what a memory system retrieves for sensitive topics — as a defensive primitive.

## What I Learned (plain English)

- Memory systems are a **write-once, poison-forever** surface: one interaction is enough, and the store itself never needs to be compromised.
- Retriever choice doesn't save you — anchors exploit topic-level cues that any decent retriever will match.
- Selective poisoning (non-target queries unaffected) means you can't detect the attack by watching for weird outputs; you have to audit the retrieval layer itself.
- Defensive implications: treat memory stores as untrusted input, rate-limit what gets persisted, and periodically verify that retrievals for sensitive topics aren't being steered.
