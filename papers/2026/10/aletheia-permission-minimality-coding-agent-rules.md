# Aletheia: Permission-Minimality Testing for Coding-Agent Rules

**Authors:** Jieke Shi, Yuchen Chen, Junda He, Yue Liu, David Lo
**Institution:** Singapore Management University (David Lo's group)
**Published:** September 30, 2026
**Link:** https://arxiv.org/abs/2609.39678
**Code:** Not stated in abstract

---

## Problem Statement (plain English)

Coding agents read repository "instruction files" (e.g. `AGENTS.md`, `.cursorrules`, `CLAUDE.md`) that tell them how to behave in a repo. Those files can also carry injections: a malicious rule can quietly ask the agent for credentials or ask it to move data off-box *while the agent still produces a correct-looking patch*. Existing defenses mostly look at the text of rules. Aletheia instead asks a behavioral question: **does this rule request more authority than the task actually needs?** Over-requested authority is a strong signal of a hidden malicious goal.

## Methodology (technical)

Aletheia frames permission testing as a *dispensability* problem:

1. **Typed authority language** — it translates everything a rule requests (file reads/writes, network egress, credential access, shell execution, etc.) into a typed language so permissions can be reasoned about and synthesized.
2. **Sandbox synthesis** — from that typed model it synthesizes *executable* sandbox configurations.
3. **Baseline vs. one-permission-removed runs** — it runs the unchanged rule + task under (a) full permissions and (b) independent restrictions that each remove exactly **one** permission.
4. **Dispensability witness** — if the task still passes its functional tests under strictly reduced authority, that permission is *dispensable*, i.e. the rule asked for something the task didn't require. Aletheia interprets witnesses against task context to diagnose suspicious requests.
5. **Formalization** — the paper formalizes the synthesis procedure and the conditions linking witnesses to what was actually enforced.

## Key Results

- On a shared refactoring task, Aletheia **executes and detects all 314 AIShellJack attack inputs**.
- **Zero alarms** on five benign templates.
- Across **80 manually verified benign GHAgentFiles rules**, only **3 false positives (3.75%)**.

## What's Novel

- Shifts agent-rule security from *static text inspection* to *least-privilege differential testing* — detection comes from behavior under reduced authority, not keyword matching.
- Introduces a formal notion of a **dispensability witness** to prove a rule over-requests authority.
- Notably resistant to the Pretext-style attack (payload in natural language): Aletheia doesn't need to parse intent, only to observe that the extra permission isn't needed.

## My Connection (to Manny's work)

- Direct red-team complement to inst-target checks: Aletheia is a *detector* for exactly the injected-rule class of attack used in agent skill/instruction poisoning.
- The "remove one permission, re-run tests" loop is a reusable harness pattern for testing any agent tool/skill: instrument least privilege, then prove over-request.
- Pairs cleanly with ActionGuard (execution-boundary authorization) and Pretext (skill-scanner evasion) from the same day — three complementary angles on the same skill/rule trust problem.

## What I Learned (plain English)

You can catch a lying instruction file without understanding what it "meant." Give the agent full permissions, run the task; then remove one permission at a time and run it again. Any permission the task *didn't* need is a red flag — the rule asked for something it had no business asking for. That's cheap, behavioral, and hard for an attacker to talk their way around.
