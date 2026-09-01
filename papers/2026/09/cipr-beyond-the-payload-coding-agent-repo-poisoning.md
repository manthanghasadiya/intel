# Beyond the Payload: How User Invocation Shapes Coding Agent Vulnerability to Repository Poisoning

- **Paper:** https://arxiv.org/abs/2608.30686
- **Authors:** Fukang Zhu, Binbin Zhao, Ruixiao Lin, Ping He, Tianyu Du, Shouling Ji
- **Published:** 2026-08-31
- **Code:** Not specified in abstract

## Problem Statement (plain English)

Coding agents bootstrap projects from third-party repos whose integrity can't be assumed. Prior repo-poisoning research focused on what *attackers* control (injection, disguise). This paper asks the neglected question: how much does the *user's own prompt* — what task they delegate, how they phrase it, which skills/rules they load — change whether a poisoned repo actually bites?

## Methodology (technical)

Introduces **CIPR (Coding In Poisoned Repos)**, the first benchmark that systematically varies **Prompt-Level Configurations (PLCs)** in poisoned real-world repositories:

- **1,920 instances** across **20 repositories**
- **4 task types** (e.g., implementation, test execution)
- **3 prompt styles** grounded in social-media phrasing
- **3 skill/rule conditions**

Measures **attack success rate (ASR)** and **agent alert rate (AR)** using automated runtime and trace-based oracles (no LLM judges).

## Key Results (with numbers)

- **Task type drives up to a 4.5x difference in ASR** — vulnerability is highly context-dependent, not a static property of the poisoned repo.
- **Test-execution tasks form a silent attack surface:** high ASR with low AR (agents execute malicious code without raising alarms).
- **Underspecified prompts reduce ASR** by truncating execution depth.
- **Noisy prompts suppress alerts** — extra noise makes malicious content less conspicuous.

## What's Novel

First to treat the *user's invocation choices* as a first-class variable in coding-agent poisoning, with a benchmark built to measure it. The finding that test-execution tasks are a silent kill surface is new and directly actionable.

## My Connection (to Manny's work)

Manny's red-team work should treat "the user's prompt" as part of the threat model, not a constant. When we evaluate agents against poisoned repos, we should sweep PLCs (task type, phrasing, loaded skills) the way CIPR does — otherwise we'll over- or under-estimate real-world ASR by up to 4.5x. Also: test-execution flows deserve priority audit since agents don't alert on them.

## What I Learned (plain English)

Repo poisoning isn't a fixed property of the malicious payload — the same poisoned repo is dangerous or harmless depending on how the human asks the agent to use it. That means defenders can't just scan repos; they have to constrain *how* agents are invoked (especially test execution) and treat noisy, underspecified prompts as risk amplifiers or suppressors respectively.
