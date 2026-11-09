# MemoryDojo: Benchmarking Poisoning Through LLM Agent Memory

**Authors:** David H. Yang, Blake Bullwinkel, Amanda Minnich, Mohammad Mohammadi Amiri, Giorgio Severi
**Institution:** Not stated on the abstract page; several authors (Bullwinkel, Minnich) publish with Microsoft's AI Red Team
**Published:** Submitted Nov 6, 2026 · announced Nov 9, 2026
**Link:** https://arxiv.org/abs/2611.09721
**Code:** Not linked in the abstract page at time of writing

---

## Problem Statement

Agents increasingly keep memory so they can work across sessions — notes, summaries, retrieved facts, user preferences. That memory is written from content the agent encountered, which includes content an attacker controlled. A malicious instruction seen in session 1 can be stored, later retrieved, and acted on in session 12, long after the original context is gone. Because memory systems differ wildly in how they write, compress and retrieve, nobody had measured whether the poison actually survives the round trip, or which architectures are worse.

## Methodology

MemoryDojo is a benchmark, not a defense. It builds a lifecycle view:

- **155 multi-session, long-horizon trajectories** across four domains, each requiring information to carry forward across sessions.
- **Five different memory systems** supported, so the same poison can be pushed through different write/retrieve pipelines.
- **Lifecycle tracing:** the benchmark instruments the path from ingestion → storage → retrieval → action, so you can see where a poison is preserved, mutated, or lost.
- **Memory-aware poison optimization:** a search procedure that adapts the injection to a target memory architecture, rather than assuming one universal payload.

The poisoning is indirect: the attacker supplies content the agent later processes, not direct access to the memory store.

## Key Results

- **All five memory systems are susceptible** to memory poisoning — there is no architecture in the set that shrugs it off.
- **Downstream effects vary substantially** between systems, meaning a payload tuned for one memory design transfers poorly to another.
- **Optimized poisons beat generic ones**, and the optimization gains differ by architecture, which is itself a signal about how each system compresses or filters.
- Measured effect is on *subsequent agent behavior*, not just retrieval accuracy — the benchmark scores whether the agent acts on the poison.

## What's Novel

Prior agent-poisoning work (memory injection, RAG corpus poisoning, tool poisoning) mostly tested a single memory implementation or a single turn. MemoryDojo treats memory as a *pipeline with variants* and measures persistence and behavioral activation across them. The memory-aware optimizer is the interesting half: it turns "is this system poisonable" into "how much attacker effort does each architecture cost."

## My Connection

Manny's agent work keeps long-lived state — memory files, skills, session notes, tool outputs. This paper gives an argument for treating memory writes as untrusted input with provenance, and a benchmark harness you can reuse to test your own harness. The architecture-dependent variation also means "we use a different memory backend" is not a mitigation claim.

## What I Learned

Persistence is the product. A prompt injection that dies with the context window is a nuisance; one that gets written to memory and re-read a week later is an implant. If your agent can write to memory from untrusted content without provenance tracking, you have a persistence mechanism with no EDR coverage — and the fix is architectural (taint the write, re-validate on retrieval), not a better system prompt.
