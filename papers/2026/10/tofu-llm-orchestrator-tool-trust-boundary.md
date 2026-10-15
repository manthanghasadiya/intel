# ToFu: Exploiting and Fuzzing the LLM–Orchestrator–Tool Trust Boundary

**Authors:** Md Nazmul Islam Ananto, Pierre Prevost-Avenel, Colin Smith, Kyungtae Kim, Taegyu Kim, Arslan Khan
**Institution:** Pennsylvania State University (and collaborators)
**Published:** October 9, 2026 (arXiv:2610.13651v1, cs.CR) — 33 pages, 4 figures, 16 tables
**Link:** https://arxiv.org/abs/2610.13651
**Code/artifacts:** Referenced in paper appendices (sanitizer corpus, evaluated repository list, verbatim LLM prompts); "Repeat-After-Me" transcript included

---

## Problem Statement (plain English)

When an LLM agent calls a tool (via MCP or a similar interface), the agent's *model-generated arguments* flow straight into that tool. Tools usually publish metadata describing what their inputs should look like — e.g., "this is a file path," "this is a URL" — but almost nothing forces the tool to actually *enforce* those rules at runtime. So the tool silently trusts whatever the model hands it. If the model can be steered (by a malicious document, a poisoned tool description, a shared workspace file, or a simple prompt pattern), it can make the tool do things the tool was never meant to do — read arbitrary files, make server-side requests, or execute commands. The paper formalizes this as the **LLM–orchestrator–tool trust boundary** and asks: how often do real-world tools actually fail to enforce their own declared semantics?

## Methodology (technical)

ToFu is a **tool-aware fuzzing framework** with three moving parts:

1. **Semantic security-check derivation.** ToFu reads each tool's existing metadata (parameter names, types, descriptions) and derives the *implied* security invariants — e.g., a parameter described as a relative path should not escape the working directory. It pairs this with a **CWE-guided sanitizer corpus** so that checks map to known weakness classes (path traversal, SSRF, command injection, etc.).
2. **Per-tool specialization + harness generation.** Using runtime introspection, ToFu specializes generic checks to each individual tool and **auto-generates fuzzing harnesses and semantic oracles** — oracles that judge whether the tool violated its own contract (a *semantic* violation), not merely whether it crashed.
3. **CWE-guided fuzzing and verification.** It fuzzes the tool with model-style arguments, then sanitizes outcomes to confirm true positives — filtering out crashes that are not security violations and surfacing violations that crash-based testing would miss. The authors manually verify findings.

They also demonstrate **"Repeat-After-Me"**: a prompt pattern that asks the model to simply *echo back* a structured tool call (attacker-specified), which is then executed — turning a benign model into an unwitting tool-call launcher.

## Key Results (with numbers)

- Evaluated **273 open-source MCP servers** containing **3,405 tools**.
- Verified **253 semantic security bugs** corresponding to CWE weakness classes, affecting **343 tools across 118 of the 273 servers** (~43% of servers).
- Weakness classes found: **path traversal, server-side request forgery (SSRF), data exfiltration, command injection, reflected output, and log injection**.
- **Repeat-After-Me** reproduced attacker-directed tool calls in **six agent frameworks**: LangGraph, AutoGen, Google Agent Development Kit (ADK) 2.0, AWS Strands Agents, Microsoft Agent Framework, and CrewAI.
- Caveats: the work is a **preprint** (not peer-reviewed), evaluates **fixed software versions**, and does not establish that *every* deployment of a listed server/framework is vulnerable.

## What's Novel

- Reframes agent security as a **metadata-vs-enforcement gap** in tools, rather than only prompt injection at the model.
- **Automatically derives** semantic security checks and fuzzing oracles from tool metadata + a CWE-guided corpus — no hand-written per-tool test cases.
- A reusable **benchmark-scale empirical count** of real MCP tool bugs (the largest such corpus to date).
- The **Repeat-After-Me** demonstration: a trivially simple prompt pattern that converts "model can request a tool" into "attacker can choose the tool call" — reproduced across six major frameworks.

## My Connection (to Manny's work)

This is a direct red-team target. Manny builds and attacks agent harnesses — ToFu gives (a) a repeatable methodology to fuzz any MCP server's tools by deriving checks from their own metadata, (b) a concrete corpus of 118 vulnerable servers to study for exploit patterns, and (c) a cross-framework injection primitive (Repeat-After-Me) to test whether a given orchestration layer enforces tool-call authorization *independently of the model*. It also gives the cleanest framing for a deliverable: "the model's *ability to request* an action ≠ the system's *authority to carry it out*."

## What I Learned (plain English)

Tools describe what they expect but rarely enforce it — and agents will happily feed them whatever an attacker engineers. You cannot fix this by making the model "behave," because the bug lives in the tools and the orchestrator. The fix is defense at the *execution* layer: validate arguments, constrain file/network access, and enforce authorization at the point the tool actually runs, independent of the LLM. Treat every model-generated argument as untrusted input, exactly as you would an HTTP request from the public internet.
