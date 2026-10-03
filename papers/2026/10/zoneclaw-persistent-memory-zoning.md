# ZoneClaw: Mitigating Persistent Memory Attacks via Memory-Zoning in Computer-Use Agents

**Authors:** Haokai Ma, Chieh Lin, Yupeng Qiu, Ee-Chien Chang (National University of Singapore)
**Institution:** National University of Singapore (NUS)
**Date:** Submitted 30 Sep 2026 · Announced 2 Oct 2026
**Link:** https://arxiv.org/abs/2610.00450
**Code:** Available (linked in abstract, "this https URL")

---

## Problem Statement

Computer-use agents (CUAs) like OpenClaw increasingly run as long-lived assistants that keep a persistent workspace: files holding user instructions, system summaries, and *external claims* are all auto-reloaded into context at the **same privilege level**. The consequence: **remembering a claim confers authority over later behavior.** An attacker who controls only benign-looking external content can get the agent to *write down* attacker-favored claims during a legitimate task, and those stored claims then silently govern future tasks the attacker never touches.

This extends the classic "malicious context → malicious response" injection into a longer, nastier chain: **malicious context → memory injection → malicious execution**, turning it into a cross-environment, persistence-style threat.

## Methodology

The key insight is that **persistence and authority are conflated** in flat workspace memory. Prior defenses intervene *before* content enters memory or *at the action it later induces* — neither governs whether stored content is *allowed to guide action*.

**ZoneClaw** replaces flat workspace memory with **hierarchical trust zones carrying explicit authority levels**:

- External claims can persist freely, but only into a **low-trust zone**.
- Such claims acquire action-guiding authority **only by crossing an explicit authority boundary**, i.e. by being promoted — and promotion is **cross-checked against zones the attacker cannot directly write**.
- The boundary is enforced by **role-specific processes of asymmetric privilege**, so **no single process both ingests external content and acts outward** (a capability-separation property).

This restructures the agent from a monolithic reasoner into a least-privilege pipeline, analogous to privilege rings in an OS.

## Key Results

Evaluated across **4 attack scenarios × 2 injection settings × 4 backbones**:

- Attack success rate (ASR): **372/480 → 6/480** (≈77.5% → 1.25%).
- Utility retained: **458/480** trials.
- Remains effective against **some defense-aware attackers**.
- Notably, attacker claims still persist in low-trust memory — they just rarely cross the authority boundary. This shows ZoneClaw **withholds authority rather than refusing to learn from the environment** (it doesn't degrade into a "reject all memory" defense).

## What's Novel

1. **Names and models "persistent memory attacks"** as a distinct, cross-environment threat class rather than a one-shot injection.
2. **Separates persistence from authority** — a control-plane distinction, not a content filter.
3. **Capability separation**: no process that reads untrusted input can also act outward, enforced via asymmetric-privilege role separation.
4. **Promotion gate** verified against attacker-unwritable zones, defeating memory-poisoning-style self-endorsement.

## My Connection (Manny's Work)

This is the memory/context tier of the agent control plane, and it pairs directly with the provenance and capability-enforcement papers already in the pipeline (ToolFence, PACE, AGATE). Where those gate *tool calls*, ZoneClaw gates *whether stored knowledge may steer behavior* — the missing layer for long-running assistants. For red-teaming: the memory-promotion boundary is a concrete new target — can you get a claim promoted without writing the high-trust zone? The claim about "defense-aware attackers" being only partially covered is the interesting seam.

## What I Learned (Plain English)

If an agent writes something down, treat that as a security event — because persisting a fact is what makes the agent obey it later. The fix isn't "don't remember external stuff"; it's "remember it, but don't let it vote until it survives a check from a source the attacker can't tamper with." Memory is an authorization boundary, not just a scratchpad.
