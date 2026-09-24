# Ajar: Measuring Open Privilege in Agent Defenses

**Authors:** Reshabh K Sharma, Linxi Jiang, Shuo Chen, Zhiqiang Lin | **Published:** Sep 22, 2026
**Link:** https://arxiv.org/abs/2609.26900
**Code:** Artifact attached to AgentDojo (per paper; not separately linked in listing)
**arXiv ID:** 2609.26900v1

---

## Problem Statement

A language-model agent acts through the tools it's given, and the data it reads while working can redirect what it does with those tools. A growing class of defenses therefore sits **between the agent and its tools**, enforcing access control, information-flow or isolation at that boundary. But these defenses are evaluated on agent-security benchmarks built around **indirect prompt injection**, which grade a defense on only two axes: *how far it brings attack success down* and *how much benign utility it preserves*. A defense can score well on both **while leaving open** a transfer, a deletion, or a broad read that **no task ever needed**. Nobody was measuring that.

## Methodology (technical)

- **Ajar measures "open privilege" directly**, by attaching to an **already-existing** agent-security benchmark and reusing its **tasks, tool schemas, reference solutions and goal states** (no new benchmark needed).
- For each **benign task**, Ajar builds a set of **candidate tool calls the task does not need** — allowing any of them means privilege has been left open.
- These candidate calls are presented to the defense **at every point where the agent could act**, so a call is "open" only if the defense would permit it there.
- Evaluation: Ajar is attached to **AgentDojo**, turning open privilege into a **third axis** alongside attack success and benign utility.
- Tested against **five defenses**: Progent, CaMeL, AC4A, Permission Assistant, and **Claude Code's Auto mode**.

## Key Results (with numbers)

- The five defenses **leave widely different amounts of privilege open** — they are not interchangeable on this axis even when their headline injection scores look similar.
- **Two defenses leak by almost the same amount yet differ widely in the benign tasks they finish** — i.e., the amount of open privilege does *not* predict how much useful work a defense enables. (This is the core finding: the third axis is genuinely independent.)

## What's Novel

- Introduces **open privilege** as a first-class, measurable property of agent defenses, distinct from attack success and utility.
- Shows the metric can be computed **on top of existing benchmarks**, so the barrier to adoption is low.
- Demonstrates that "best injection score" and "best utility" do not identify the safest *or* the most usable defense — the two-leak-similar / different-utility result breaks the assumption that these defenses are simply better or worse versions of each other.

## My Connection (to Manny's work)

When Manny evaluates or builds an agent defense (permission assistants, runtime authorization, sandboxed tool proxies), attack-success benchmarks alone are insufficient evidence. Ajar is the template for asking the harder question: **what did this defense allow that no task required?** It's also a diagnostic Manny can point at his own harness to surface over-permissive tool grants before an adversary does.

## What I Learned (plain English)

A defense that blocks every attack you throw at it can still be handing out permissions nobody asked for — like a guard who stops known intruders but leaves every side door unlocked. Measuring "what should this agent *never* be able to do" is a separate job from measuring "did it get hacked," and doing that measurement is easier than it sounds because you can bolt it onto benchmarks you already run.
