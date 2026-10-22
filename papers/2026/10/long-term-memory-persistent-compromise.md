# Combining Long-Term Memory and LLMs Comes at a High Cost: From Prompt Injection to Persistent Compromise

**Authors:** Pedro H. Barcha Correia, Matheus Azevedo de Sá, Paulo H. Gomes de Senna, Giovanni Mandel Martignago, Victor Takashi Hayashi, Charles Christian Miers, Marcos A. Simplicio Jr
**Published:** Submitted October 20, 2026; announced in the cs.CR new listing for October 22, 2026 — arXiv:2610.28810 [cs.CR]
**Link:** https://arxiv.org/abs/2610.28810
**Subjects:** cs.CR; cs.AI; cs.CL; cs.LG
**Institution:** Not stated in arXiv metadata; author group is Brazil-based academic (the affiliations correspond to Brazilian university security/engineering groups).
**Artifacts:** Datasets and attack artifacts released by the authors.

---

## Problem Statement

Classic prompt injection is bounded by the session: the malicious instruction is in the context window, it influences that conversation, and it is gone when the window clears. Persistent memory breaks that assumption. Once ChatGPT, Claude and Gemini began carrying memory across conversations — personal memories, project instructions, saved skills, organisation-level instructions — the attack surface stopped being "one chat" and became "everything this account will ever do again."

The paper's claim is that this is not a feature-specific bug but a general and under-mitigated vulnerability class it calls **memory injection**, and that the resulting compromise is *persistent*: a single successful injection can keep steering the model in fresh, clean sessions long after the poisoned conversation is closed, and in at least one platform it can spread between users inside an organisation.

## Methodology

1. **General threat model.** Rather than cataloguing per-product quirks, the authors derive an abstract model of memory injection: an attacker writes content that a memory mechanism will retain, and the retention path is what carries the payload across session boundaries.
2. **Attack-goal framing against NIST.** The evaluation is organised around NIST's attack-goal taxonomy rather than ad-hoc success criteria, so results are comparable across platforms.
3. **Multi-vector injection.** The work enumerates distinct write paths into memory: **personal memories, projects, skills, organisation instructions, and MCP-connected resources** — the last being the most interesting, since an MCP server's returned content can become durable memory rather than transient context.
4. **Platform evaluation.** Live testing against three LLM-as-a-service products: **ChatGPT Instant 5.3, Claude Sonnet 4.6, Gemini 3 Flash**.
5. **Persistence verification.** Each successful injection is checked in *fresh* sessions to distinguish transient influence from retained memory.
6. **Defence derivation.** Countermeasures are derived from the threat model and from foundational security principles (separation of data and instruction, provenance, least privilege on memory writes), then checked against what the platforms actually implement.

## Key Results

- **All NIST attack goals are collectively achievable** under the proposed model across the three evaluated services — ChatGPT Instant 5.3, Claude Sonnet 4.6, and Gemini 3 Flash.
- **Persistence confirmed:** all evaluated services could be manipulated to persistently spread disinformation, generate harmful content, or simulate offline unresponsiveness — *including in brand-new sessions*.
- **Five distinct write vectors** were validated in practice: personal memories, projects, skills, organisation instructions, and MCP-connected resources.
- **Cross-user spread on Claude:** shared-memory features allow an attacker to compromise multiple users within an organisation, and to exfiltrate other users' prompts.
- **Defence gap:** of the mitigations the authors derive from their threat model, **nearly half remain unimplemented** across the target services.

## What's Novel

The contribution is consolidation plus a generalisation step. Prior work demonstrated platform-specific memory abuses (poisoning "custom instructions", planting facts in memory); this paper (a) frames them as one class with a reusable threat model, (b) makes the MCP-connected-resource path explicit as a memory write vector rather than a context vector, and (c) quantifies the persistence property properly — verifying behaviour in *fresh* sessions instead of inside the poisoned one. The Claude shared-memory multi-user compromise is the sharpest new result: memory as a lateral-movement primitive inside a tenant.

## My Connection

This is the "blast radius" paper for everything else in the agent-security stack:

- **MCP trust boundary:** if an MCP server's output can be promoted into durable memory, then every MCP server a user installs is a potential long-lived persistence mechanism — server review has to cover *what it can write to memory*, not just what tools it exposes.
- **Agent gateways and enterprise deployments:** the shared-memory cross-user path is the enterprise-relevant one. Any org enabling shared projects/skills has a single-user-compromise → multi-user-compromise chain.
- **Detection:** memory writes are auditable events. A gateway or admin console that logs "memory created / memory edited / memory source" gives you the only reliable detection point, and the paper's finding that ~half the mitigations are missing means most tenants have no such log.
- **Assessment checklist additions:** can untrusted tool output write memory? is memory shared across users? does the UI attribute each memory to its source? can a user review and purge memory by provenance?

## What I Learned

Memory turns a one-shot injection into an installed backdoor that lives in the model's beliefs rather than in its context — so it survives the thing everyone relies on for cleanup: closing the session. The practical lesson is that "clear your chat" is not a remediation step anymore, and that memory write paths deserve the same scrutiny as tool-call paths. If you cannot answer "where did this memory come from?", you cannot evict an attacker.
