# Convergent Detour Hijacking: Task-Preserving Resource Amplification in Skill-Based LLM Agents

- **Paper:** [arXiv:2608.12273](https://arxiv.org/abs/2608.12273)
- **Authors:** Junliang Liu, Ruoyu Li, Wenxin Tang, Jingyu Xiao, Zhenyu Liu, Jingheng Xu, Laizhong Cui
- **Institution:** Shenzhen University (Laizhong Cui's group; per prior publications)
- **Published:** 2026-08-12 (v1)
- **Subjects:** cs.CR, cs.AI
- **Code:** Not listed on the arXiv page.

## Problem Statement (plain English)

Skill-based LLM agents (like Claude Code skills or MCP skill registries) work by progressive disclosure: a third-party skill has a short natural-language **description** that the agent reads when choosing what to use, and a longer **instruction body** that the agent reads only after selection. That design gives an untrusted skill publisher two separate control points — the description steers selection, the body steers planning. Prior work attacked these points in isolation (selection manipulation, malicious instructions, tool-chain resource amplification), but nobody had shown how to compose them into one reliable, end-to-end attack. This paper asks: can a malicious skill hijack the *trajectory* — wasting massive compute and time — while still letting the task complete correctly, so nothing looks wrong?

## Methodology (technical)

- **Attack model:** text-only, runtime-independent — no exploit of the agent harness, just the skill files' description + body.
- **Stage 1 — selection (semantic cover):** the malicious description is written to be maximally relevant to common tasks, so it wins the selection step (and drags in an attacker-controlled "coordinator" skill).
- **Stage 2 — planning (fabricated dependencies):** the aligned body reuses that same rationale to invent plausible dependencies, recruiting *benign* skills into a bounded detour — a longer, costlier route that still terminates.
- **Stage 3 — re-entry:** the trajectory returns to the original route so the task completes and outcome metrics look normal.
- **Evaluation:** 491 held-out tasks, single-task and multi-turn conditions, across multiple LLM backends (headline numbers on DeepSeek-V4-Pro).

## Key Results (with numbers)

- Coordinator skill selected in **80.02%** of tasks on DeepSeek-V4-Pro.
- Among coordinator-hit runs that completed: token consumption **+66.91%**, end-to-end execution time **+92.45%**.
- Aggregate task completion remained comparable to clean runs — the attack is *task-preserving* by design.

## What's Novel

- First end-to-end composition of the selection and planning control points (CDH), where prior work treated them separately.
- The attack is deliberately **outcome-preserving**: it doesn't break the task, it amplifies cost. That makes it invisible to "did the agent finish correctly?" checks and to most agent-monitoring dashboards.
- Demonstrates the attack works across multiple backends and in multi-turn settings — not a single-model artifact.

## My Connection (to Manny's work)

This is the supply-chain attack on skill-based agent systems (MCP-style) that doesn't need a single malicious tool call — it just makes the fleet slower and more expensive while looking perfect. For Manny's red-team work: an instant detection play is to baseline token/step cost per task class and alert on detours; for offensive work, CDH is a low-risk persistence/covert-resource-burn technique that evades outcome-based guardrails.

## What I Learned (plain English)

Correct task completion is not evidence of a clean trajectory. When agents pick tools from untrusted registries, the selection description and the instruction body are two separate attack surfaces that can be chained: win selection with relevance, then use the body to fabricate "necessary" detours. Monitoring agents by results alone (did the task succeed?) is blind to this — you need cost- and trajectory-level telemetry. Also: resource amplification is a real, stealthy objective on its own (energy, quota, API bills), not just a side effect of malicious tool use.
