# Chaining Skills to Hijack LLM Agents (APEX)

**Authors:** Tian Dong, Zixuan Ma, Haodong Zhao, Huaien Zhang, Shaofeng Li, Hao Chen
**Institution:** (multi-institution; see paper for affiliations)
**Published:** Oct 1, 2026
**Link:** https://arxiv.org/abs/2610.01564
**Code:** See paper's "Links to Code" (arXiv listing)
**Venue:** arXiv:2610.01564 [cs.CR]

---

## Problem Statement (plain English)

Modern LLM agents "improve" by loading **skills** — reusable, often open-source tool/procedure bundles (think MCP tools or skill plugins). When an agent handles a task it typically invokes several skills in sequence, and *information produced under one skill is allowed to guide the next*. That handoff is the soft spot: an attacker who controls a single upstream skill can inject a false claim that travels downstream and makes the agent perform an action the user never approved.

The paper asks: can an attacker deliberately construct *chains* of skills that route a target action through legitimate-looking intermediate state, defeating the natural assumption that per-skill vetting is enough?

## Methodology (technical)

- **APEX** is the attack framework. Given (a) a user task and (b) an attacker-selected action, it constructs and *refines* adversarial skill chains tailored to both.
- **Key insight:** an *agent-written record of genuine task progress* is a trusted-looking artifact. An upstream skill induces the agent to write a progress/approval record; a downstream skill reads that record and treats it as proof of user approval, then executes the attacker's action.
- Because the record is created by the agent about a *real* task, generic "is this malicious?" checks and static skill scanning tend to pass it.
- **Evaluation:** four targeted-action families across six models on **SkillsBench**.
- **Baselines/ablations:** the same malicious workflow merged into a *single* skill (to isolate the value of chaining); and a prompting defense that asks the agent to check skill-produced files against the original request.

## Key Results (with numbers)

- Full chains induced the attacker-selected action in **512 / 690 attempts = 74.2%** across four action families and six models.
- On **GPT-5.4**, the full chain succeeded in **84.3%** of attempts, versus **17.4%** when the workflow was collapsed into one skill — chaining roughly *5×* the success rate.
- The prompting defense lowered GPT-5.4 targeted-action success from **84.3% → 59.1%**, but collateral damage was severe: the verifier test-pass rate across **72 benign native-skill tasks fell from 86.7% → 56.3%**.

## What's Novel

1. Attacks the **inter-skill handoff**, not a single artifact — most prior work scans one tool/skill in isolation.
2. Weaponizes the agent's **own task-progress recordkeeping** as the trust-propagation mechanism (a "false claim of approval" riding on genuine work).
3. Quantifies that **decomposing** the attack across skills (vs. one merged skill) is what makes it succeed.
4. Shows the obvious textual defense (check outputs against the original request) trades large utility loss for modest security gain — a "verifier tax."

## My Connection (to Manny's work)

- Directly relevant to MCP/skill supply-chain security and agent red-teaming: the attack surface is **provenance across a chain**, which single-tool scanners (Snyk-style, allow-lists, per-skill review) won't catch.
- Pairs naturally with the PACE paper (provenance-aware capability enforcement) — PACE is essentially a defense against exactly this class of influence-path hijack.
- Useful for building a demo: "two innocuous skills, one malicious outcome," ideal for a content piece or a client walkthrough.

## What I Learned (plain English)

The danger isn't a poisoned skill — it's the *seam* between two honest-looking ones. If your agent lets one step's output authorize the next step's action, an attacker can split a single bad action into two steps that each look fine. And the fix is hard: asking the agent to "double-check against the original request" catches some attacks but wrecks normal task performance. The real fix has to be structural — track where each decision's authority came from and refuse actions whose authority traces to unverified provenance.
