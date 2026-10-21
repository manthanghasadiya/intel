# ClawMMU: Protection-Coherent Virtual Memory for Stateful LLM Agents

**Authors:** Jinhao Hu, Bardia Mohammadi, Mofasshara Binte Rafique, Lars Henning Klein, Akhil Arora, Ashvin Goel, Laurent Bindschaedler
**Institution:** Not listed on the arXiv abstract page (work presented at the AgenticOS workshop)
**Published:** October 20, 2026 — **AgenticOS Workshop @ NeurIPS 2026** (non-archival), 6 pages, 1 figure, 1 table
**Link:** https://arxiv.org/abs/2610.28093
**Code:** Not linked in the abstract

---

## Problem Statement

Agents keep long-term memory, and most of that memory arrives from untrusted sources: web pages, tool results, retrieved documents, other agents. Agent harnesses — the software that runs the model — record *what* the memory says but not *who may act on it*. The result is the core failure of every indirect prompt injection: an instruction hidden in a fetched page is treated exactly like an instruction from the user, because at the point of context assembly there is no difference between them.

The framing is deliberate and OS-flavored: this is a **protection** problem, not a content problem.

## Methodology

ClawMMU borrows the shape of a memory management unit. Every piece of memory is labeled at ingress with four attributes:

- **Owner** — on whose behalf the content was created
- **Scope** — where it is valid
- **Origins** — the provenance chain of where it came from
- **Authority** — what it is permitted to influence

Those labels are then **checked at four boundaries**:

1. When content **enters the model's context**
2. When content is **written to storage**
3. When content **returns from storage**
4. When content **drives a tool call**

The central rule is a one-liner: **information may rise in authority, but authority never lifts itself.** Something can be promoted (a user explicitly endorses a fetched document, an operator approves an action), but no piece of content can promote *itself* by asserting that it is trusted.

Because the checks are on **provenance, not content**, an attack does not need to be recognized as an attack. A poisoned page is stopped because its origin label does not permit tool-call authority — not because a classifier decided the text looked malicious.

## Key Results

- The design converts memory attacks into **classic protection faults**, which the harness can stop **even when the model itself is fooled**. This is the important property: the defense does not compete with the model's judgment.
- The enforcement points are structural (four boundaries), so coverage does not depend on enumerating attack strings.
- Reported as a 6-page workshop paper with one figure and one table — this is an architecture and framing contribution, not a benchmarked system with attack-success numbers in the abstract.

## What's Novel

- **Authority as a first-class memory attribute.** Existing memory-safety work zones memory (separate stores, TTLs, trust tiers per document) but rarely models *what a memory item is allowed to do* as distinct from *where it lives*.
- **The non-self-elevation rule.** Stated as an invariant, it is checkable and it kills the entire class of "the document says to ignore previous instructions" attacks without needing to detect the phrase.
- **Checking at read *and* write *and* tool-call time.** Most schemes check at ingress only, which misses the case where benign content is later recombined into an authoritative context.

## My Connection

This is the mechanism the carryover-attack paper says is missing. 2610.28425 shows that memory-planted injections re-fire across sessions with no decay and concludes that runtime-layer controls are required; ClawMMU is a concrete runtime-layer control with a stated invariant.

For Manny's Hermes setup the mapping is close to literal:

- **Origin labels** on anything written to memory by a web fetch, tool result, or inbound message — and a hard rule that such content can never carry tool-call authority.
- **A tool-call-time check** (not just prompt assembly) that asks "does the authority label on the content that motivated this call permit it?"
- **Explicit elevation events** — user confirmation, operator approval — as the only path to higher authority, logged as such.

The `claw`-prefixed name and the harness framing will read as a direct comment on OpenClaw-style computer-use agents, which makes it a natural companion piece to the ZoneClaw memory-zoning paper from earlier this month: ZoneClaw partitions memory by zone, ClawMMU gives each zone an authority lattice.

## What I Learned

Content-based defenses to prompt injection are structurally losing, because the defender must recognize the attack and the attacker only needs to look benign. Provenance-based defenses flip that: the label is attached when content arrives, so the question "may this drive a tool call?" has an answer that does not depend on reading the payload. The one-sentence version worth remembering: **trust should be a property of where a thing came from, not of how convincing it sounds.**
