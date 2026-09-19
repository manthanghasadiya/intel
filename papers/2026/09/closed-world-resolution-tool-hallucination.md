# Closed-World Resolution Against Tool Hallucination in LLM Agents

**Author:** Laxmipriya Ganesh Iyer
**Institution:** Independent (single-author)
**Date:** Submitted 16 Sep 2026 (announced in the Fri, 18 Sep 2026 cs.CR batch)
**Link:** https://arxiv.org/abs/2609.19425
**Code / benchmark:** Hallucinated-Tools Benchmark (HTB), versioned and released for cross-resolver comparison

---

## Problem Statement (plain English)

Tool-augmented agents have a failure mode nobody's defense actually covers: the model calls a tool **that does not exist**, or passes arguments **no schema declares**. Existing defenses split into two camps, and both quietly assume away the problem:

- **Tool selection** methods try to pick the *right* tool from a known set.
- **Gating** methods constrain what an agent may do *with real tools*.

Both presuppose the emitted call refers to a real tool in the first place. A hallucinated call is, by construction, not a "decision" any gate made — so no gate can reject it. The paper calls this a structural blind spot, not a bug in any one system.

## Methodology (technical)

- **Taxonomy H1–H5** for tool hallucination on a single registry surface (fabricated tools, malformed args, borrowed-but-invalid schema, etc.).
- **The Resolution Rung:** a training-free, closed-world resolver that does two things — registry membership check plus a signature check. The paper's point is not what it computes but **where it must sit**: it proves hallucination defense must precede any causal gate, because a hallucinated call never passes through the gate's decision at all.
- It characterizes one irreducible residue: "borrowed arguments" that are schema-indistinguishable from a valid call.
- **Empirical measurement:** ten hosted models, two invocation surfaces (constrained tool-calling vs. unconstrained raw JSON), counting genuine hallucinations. It also measures model scale as a variable.
- **MCP extension:** merges several MCP servers into one namespace and derives a second taxonomy (M1–M5) covering collisions and shadowing that no single registry can express.

## Key Results (with numbers)

- **322 genuine hallucinations** measured across ten hosted models.
- Fabricated-tool calls concentrate on the **unconstrained raw-JSON surface (34 vs. 3)**.
- **Scale does not help:** a **675B model matches a 7–8B model** in hallucination behavior.
- On the **live MCP surface: 154 hallucinations**, including from **frontier models that were clean on the single-registry surface** — collisions and shadowing are structural to the merge.
- Releases the versioned **Hallucinated-Tools Benchmark (HTB)** so any resolver is comparable across submissions.

## What's Novel

1. Reframes tool hallucination as an **ordering problem**, not a classification problem: no gate can reject what was never a decision, so resolution must be structurally *prior* to gating.
2. Proves the irreducibility of a small residue class ("borrowed args"), bounding what any resolver can achieve.
3. Shows **model scaling is not a defense**, killing the lazy "bigger model fixes it" assumption.
4. Extends the problem to **MCP namespace merging**, where a single-registry resolver is provably insufficient — a genuinely new attack surface tied to how agent runtimes actually compose servers.

## My Connection (to Manny's work)

This is directly actionable for MCP/agent red-teaming and harness hardening:

- The **M1–M5 merge taxonomy** is a checklist to test multi-server MCP setups — namespace collision and shadowing are the kind of thing that only bites in real composite deployments.
- "Resolution before gating" is an architectural rule to enforce in any agent runtime whose safety story rests on a policy gate.
- The **HTB benchmark** gives a reproducible way to compare resolvers, which is useful if you build or evaluate tool-calling defenses.
- The finding that frontier models stay clean on single-registry but fail on merged namespaces is a great demonstration of why capability evals must include composition, not just isolated components.

## What I Learned (plain English)

If an LLM invents a tool that was never there, there's no safety check to "catch" it — because your check was built to decide between real options, and this call was never one of them. The fix isn't a better judge; it's putting a boring, closed-world check (does this tool exist, does its signature match) *in front of* everything else. And the moment you glue several MCP servers together, whole new ways to collide and shadow appear that no single-server registry can see — so merging servers is a security decision, not just a convenience one.
