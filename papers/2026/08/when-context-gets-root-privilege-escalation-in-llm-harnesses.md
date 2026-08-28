# When Context Gets Root: Privilege Escalation in LLM Harnesses

- **Link:** https://arxiv.org/abs/2608.27299
- **Authors:** Xingbang He, Yuanwei Chen, Yi Qian, Haiyang Wei, Ligeng Chen, Zenan Fu, Linzhang Wang, Hao Wu, Bing Mao
- **Institution:** Nanjing University (State Key Laboratory for Novel Software Technology)
- **Published:** 2026-08-27
- **Code:** Not listed

## Problem Statement (plain English)

Model providers defend against prompt injection with the *instruction hierarchy*: instructions are tagged with privilege levels (system > user > tool/web content), and the model is trained to only obey higher-level instructions. That defense assumes the levels a model sees are the levels the attacker's content deserves. But agents don't run on raw model APIs — they run in *harnesses* (Claude Code, Cursor, Kiro, etc.) that build the context for every model call. This paper shows the harness's context construction can silently *promote* attacker content to a higher instruction level — instruction privilege escalation — so the model happily follows instructions it would have refused at their true level.

## Methodology (technical)

- Attack surface: the context-assembly step of six coding-agent harnesses, where content from different sources (system prompt, user input, tool output, files) gets concatenated/relabeled into one prompt.
- Attack mechanism: attacker induces the agent to move low-level malicious content into a slot the harness treats as higher-privilege (e.g., a persistent goal, a system-style directive, a scheduled task), then the elevated content drives behavior.
- Evaluation: multi-agent mechanisms (one agent distracts/positions, another delivers the payload) targeting **13 attack objectives** spanning confidentiality, integrity, availability, and remote code execution, across **six coding-agent harnesses**.
- Tested under two execution modes: unrestricted action execution, and automatic permission review (harness asks a reviewer model before dangerous actions).

## Key Results (with numbers)

- **Unrestricted execution:** all 13/13 objectives achieved on **all six** harnesses.
- **Automatic permission review:** all 13/13 objectives still achieved on **all three** harnesses that provide that mode — the permission reviewer itself is part of the context the attacker can elevate through.
- The vulnerability reproduces via harness-provided **persistent goals** and **scheduled tasks** — i.e., persistence mechanisms survive across sessions, not just single turns.

## What's Novel

- Shifts the blame from the model to the harness: instruction hierarchy is a model-side defense, and this is the first systematic demonstration that *harness context construction* is an independent privilege-escalation primitive that defeats it — with persistence via scheduled tasks and goals.

## My Connection (to Manny's work)

Directly actionable: Manny should audit every harness in use (Claude Code configs, agent frameworks, MCP orchestration) for context-assembly paths where low-privilege content (tool output, files, fetched web content) can land in high-privilege slots — persistent goals and scheduled tasks are the sneakiest. It also means "automatic permission review" is not a sufficient control; the reviewer's context is attackable too. This pairs with the Kiro finding: repo content → agent → IDE config is the same trust-boundary collapse.

## What I Learned (plain English)

- The instruction hierarchy only protects the *model*; the *harness* decides what privilege labels content actually gets, and that assembly step is attackable.
- A "human approval" step is still in-context: if the attacker can elevate content that the reviewer reads, the approval is theater.
- Persistence features (goals, cron/scheduled tasks) are privilege-escalation gifts — they let one successful elevation survive across sessions.
