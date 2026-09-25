# Persistent Billable State: Denial-of-Wallet Attacks and Defenses in Tool-Calling LLM Agents

**Authors:** Jinqian Zhang, Haojun Xia, Shujiang Wu, Jingkun Yue, Xia Zhang, Zhangpei Cheng, Bibo Tu
**Institution:** Institute of Information Engineering, Chinese Academy of Sciences; School of Cyber Security, University of Chinese Academy of Sciences; Beihang University; State Key Laboratory of Networking and Switching Technology, Beijing University of Posts and Telecommunications
**Published:** September 23, 2026
**Link:** https://arxiv.org/abs/2609.28585
**Code:** Not linked on the abstract page
**Subjects:** cs.CR (primary); cs.AI

---

## Problem Statement (plain English)

Multi-step tool-calling agents rely on a **host runtime** to carry state between turns. When that runtime feeds a tool's output back into the model's later inputs, the model provider **meters it again** — you pay for the same bytes on every subsequent turn. An attacker who controls (or compromises) a single tool can therefore turn untrusted data into **recurring, victim-billed processing** — a denial-of-wallet attack — *without any victim credentials and without any local runtime privilege*. The victim did nothing wrong; the agent just kept paying to re-read the attacker's payload.

## Methodology (technical)

- Names the phenomenon **persistent billable state** and formalizes the host's decision about whether/how a tool return enters later billable context as the **persistent billable-state boundary**.
- Runs the **first systematic security study of the post-admission lifecycle** — i.e., what happens to a tool's output *after* it has been admitted/accepted, not just at admission.
- Derives **six denial-of-wallet attack vectors** and builds **DOW-BENCH**, an end-to-end evaluation harness exercised across **six model families**.
- Compares retention policies: raw retention, compression, and deletion — measuring both cost and task success.
- Proposes a governance "kernel": deterministic history transformation plus **four host-side invariants** bounding prompt mass, context growth, recursive opportunity, and cumulative spend *before* re-ingestion.

## Key Results (with numbers)

- Across **243 executions**, the **maximum per-session cumulative input reaches 14,293×** the session's first-call input.
- Controlled history-policy reruns isolate raw retention's contribution: retaining raw history increases **mean effective session cost by 21.2–35.9%**.
- **Compression** preserves task success on **10/12 and 11/12** history-dependent tasks (per provider), versus only **2/12 under deletion**.
- The proposed kernel **contains every recurring attack in the 123-evaluation replay corpus**; across **24 Mistral Small 4 workflows** it retains task progress while bounding spend.

## What's Novel

Everyone reasons about agent cost as a *scaling* property; this paper makes it a *security* property. It introduces an economic denial-of-service class that needs no exploit, no credential and no privilege — just a tool that returns data the runtime keeps re-ingesting and re-billing. The formal "persistent billable-state boundary" plus the six-vector taxonomy and a containment kernel make it directly actionable rather than theoretical.

## My Connection

This is the missing category in agent threat modeling: alongside runaway actions and data exfil, there's **runaway spend**, and it's reachable by any attacker who can inject a single tool output (MCP tool poisoning, prompt-injected content in a RAG corpus, a compromised third-party tool). For content, "the agent didn't get hacked, its wallet did" is a fresh, concrete hook — and the four-invariant containment design is a clean defensive artifact to walk through. It also pairs naturally with today's approval-laundering and trace-tampering papers: integrity, authorization, and now *economics* are the three pillars of agent-runtime security.

## What I Learned (plain English)

When a system pays per-token to re-read its own conversation history, an attacker's junk data becomes a recurring bill. You don't steal anything — you just make the victim's agent keep paying to look at what you wrote. The defense isn't detection, it's a hard budget on how much untrusted content can enter later billable context.
