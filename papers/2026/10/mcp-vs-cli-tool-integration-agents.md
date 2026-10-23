# MCP vs CLI for AI Agents: A Comparative Analysis of Tool Integration Strategies for Autonomous Systems

**Authors:** Arun Malik, Nitish Talasu, Bradley Klemick, Vineet Tushar Trivedi, Prachi Shah, Sanay Devi, Junjie Zhang, Kunal Sonawane
**Institution:** Not stated in the arXiv listing (industry + academic author mix)
**Published:** arXiv:2610.30157 — v1 Wed, 29 Jul 2026 05:45:07 UTC; announced to cs.CR cross-list Oct 23, 2026
**Link:** https://arxiv.org/abs/2610.30157
**Code:** None advertised

---

## Problem Statement

An agent that can't touch anything is a chatbot. To be useful it needs an interface to files, APIs, databases, and cloud services, and the industry has split on which one it should have. One camp gives the agent a shell — it runs `git`, `curl`, `psql` like a human operator would. The other gives it MCP: a structured JSON-RPC protocol with typed tools, declared schemas, and a server that sits between the agent and the resource.

The debate has been almost entirely vibes and token-counting. CLI advocates point out MCP burns context on tool definitions and adds a hop. MCP advocates point out a shell is an unbounded capability. Neither side has produced a side-by-side on the dimensions that actually decide production deployments: access control, credential lifetime, audit, multi-tenancy, and cost at scale.

This paper asks the question directly: across 14 evaluation dimensions, which integration strategy should a production system use?

## Methodology

A structured comparative analysis rather than an empirical benchmark. The authors build out 14 dimensions and evaluate both paradigms against each, using worked examples (concrete agent tasks executed against each interface), a security assessment mapped to published compliance guidance, and an economic model.

The security work is anchored to real guidance rather than invented criteria — specifically NSA CSI U/OO/6030316-26 (formal NSA guidance) and OWASP security standards for LLM applications. The economic modeling draws on BCG and Gartner cost figures rather than self-reported vendor numbers.

The most technically substantive section is on **progressive disclosure**: instead of loading every tool schema into context up front, the agent sees a compact index and expands definitions only for tools it actually considers. This is the mechanism the authors use to argue MCP's context overhead is a solved problem rather than an architectural tax.

## Key Results

- **CLI wins on efficiency for single-user, local-only workflows.** Fewer tokens, no protocol hop, no server process; the shell is already there and the agent already knows it.
- **MCP does four things CLI structurally cannot:** fine-grained access control, credential isolation, structured audit, and multi-tenant governance. These are not efficiency trade-offs — a shell cannot express "this agent may read `customers` but not `payments`, using a scoped token that expires in 15 minutes, and log every call."
- **Progressive disclosure closes the context-window gap.** With tool definitions loaded on demand, MCP token efficiency becomes comparable to CLI while retaining the architectural properties.
- **Recommendation: hybrid.** CLI for the operator/developer-facing surface where the agent is trusted and local; MCP wherever the agent is untrusted, multi-tenant, or needs an audit trail.

## What's Novel

The contribution is not a new attack or a new protocol feature — it's the framing that ends the argument. The paper separates **efficiency** (where CLI genuinely wins and always will) from **governance** (where MCP is not competing, because a shell has no representation for a permission). Once you state it that way, "MCP is slower than Bash" stops being an objection and becomes a price.

Second novelty: treating progressive disclosure as the answer to the context objection, with the argument that the overhead was a *client implementation choice* rather than a protocol property. That reframes a decade of MCP criticism.

Third: mapping MCP's control surface to NSA CSI and OWASP explicitly, which gives practitioners a compliance-shaped argument rather than a preference-shaped one.

## My Connection

This is the citation you use when someone on the team says "why not just give the agent a shell?" — and it matters more than usual right now, because the last week of this digest has been a catalogue of what happens when the agent *does* have a shell. OpenCode's MCP servers executing code at sign-in, LiteLLM backdoored through logged tokens, x402 clients signing whatever a server asked for, TrojanForge poisoning repos the coding agent then reads.

The 14-dimension framing maps cleanly onto how you'd threat-model an agent harness: for each capability the agent has, ask whether the interface can express *who* is calling, *what* they may touch, and *that it happened*. CLI answers "no" to all three by construction. That's not a reason to ban it — it's a reason to know which of your agents are running on an interface with no policy layer, and to keep those agents on a short leash with a filesystem boundary instead.

The progressive-disclosure point is the practically useful one: if you had rejected MCP on token cost, that objection has an implementation fix and you should re-evaluate.

## What I Learned

A shell and a protocol are not two ways to do the same thing. A shell is a capability with no name; MCP is a capability with a name, an owner, and a receipt. The token overhead everyone complained about was a client-side laziness, not a law of physics — load the tool list lazily and MCP costs about what Bash costs, while still being able to say who did what.

The uncomfortable implication: if you're running an agent with shell access in anything multi-tenant, you don't have a permissions model, you have a hope. The paper's hybrid recommendation is really an admission that the right answer depends on whether you can trust the agent — which means the first question in any agent design review is not "MCP or CLI" but "is this agent trusted, and by whom."
