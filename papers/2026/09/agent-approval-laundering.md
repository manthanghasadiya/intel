# Agent Approval Laundering: Transitive Effects Beyond the Approved Invocation

**Authors:** Jinqian Zhang, Haojun Xia, Shujiang Wu, Jingkun Yue, Xia Zhang, Zhangpei Cheng, Bibo Tu
**Institution:** Institute of Information Engineering, Chinese Academy of Sciences; School of Cyber Security, University of Chinese Academy of Sciences; Beihang University; State Key Laboratory of Networking and Switching Technology, Beijing University of Posts and Telecommunications
**Published:** September 23, 2026
**Link:** https://arxiv.org/abs/2609.28586
**Code:** Not linked on the abstract page
**Subjects:** cs.CR (primary); cs.SE

---

## Problem Statement (plain English)

When a coding agent asks a human "run this command?", the human approves **one thing** — a command, or a single tool call. But that command activates a whole *workflow*: `npm install` runs lifecycle hooks and writes files; an MCP call exercises network authority; a build step touches credentials. The durable record of the approval names only the **entry invocation**, and silently omits all the downstream effects the approval actually unlocked.

The authors call this **approval laundering**: the audit trail records a benign wrapper while the real effects run outside what was reviewed. A reviewer effectively approved a closure but only saw (and only signed) the first node.

## Methodology (technical)

- Takes a **record-to-closure** view of agent approvals: bind the durable approval record to the full set of effects its workflow can exercise.
- **Formalizes closure-bound approval over six effect classes** and derives an **information limit**: two invocations that are indistinguishable in their policy-visible fields can still require *different* effect-specific decisions — so no policy that looks only at the record can guarantee the correct decision for both.
- Builds the **Approval-to-Action Security Benchmark**, which binds approval objects and decision-time metadata to post-execution evidence (i.e., checks the record against what actually happened).
- Evaluates a ladder of record enrichment: explicit fields → command semantics → decision-time metadata → effect-bound records with frozen, source-backed predictions and provenance committed *before* authorization.

## Key Results (with numbers)

- Across **111 fixed approval-object/trace pairs**, residual (unexplained) records fall from **40** with explicit fields, to **17** with command semantics, to **13** with decision-time metadata.
- Across **11 fixed-SHA executions**, the enrichment ladder reaches **zero metadata residuals**; **two exact mappings recur across three product frontends**.
- For prospective recovery, effect-bound records committing predictions + provenance before authorization achieve **0.926 macro recall** and **0.941 macro** precision on **17 prespecified holdout workflows**.

## What's Novel

The framing itself is the contribution: it reframes agent approval from "did the user click yes on this tool call?" to "does the record cover the effect closure?" It supplies a **formal information-theoretic limit** (record-only policy cannot be sound for all effects) and then shows a practical path — predict and commit the effect set *before* authorization — that empirically closes the gap. Most prior agent-approval work sizes the prompt; this sizes the **coverage** of the authorization.

## My Connection

This is the mechanism behind a whole class of "the human approved it, so it's allowed" bypasses — directly relevant to MCP tool-poisoning chains and to any agent with a human-in-the-loop gate (cf. prior digests' loopjacking and plugin4shell work). For red teaming: the attack is to get a benign-looking invocation approved whose transitive closure does the damage. For defensive content: it's the strongest argument yet that approval UIs must preview *effects*, not commands, and that the approval record must be committed before execution.

## What I Learned (plain English)

Approving a command is not the same as approving what that command *does*. Because the tools underneath add their own hook-driven side effects, a reviewer can be honest, careful, and still sign off on something they never actually saw. If you want approvals to mean anything, you have to approve the consequences, not the label.
