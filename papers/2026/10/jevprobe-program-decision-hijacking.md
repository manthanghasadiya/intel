# Nudge the Judge: Uncovering Program Decision Hijacking in Real-World Jev Applications

**Authors:** Yang Luo, Tiantian Ji, Zifeng Kang, Yucheng Wang, Yong Liu, Zhiyi Yin, Xinran Liu
**Institution:** (multi-institution; see paper) — research disclosed responsibly to Jev application maintainers
**Published:** October 13, 2026 (arXiv:2610.18612, cs.CR / cs.AI)
**Link:** https://arxiv.org/abs/2610.18612

---

## Problem Statement (plain English)

"System One"-style models (referred to as **Jev**) change how software makes decisions: instead of a fixed `if/else`, the program maps open-ended, messy context (a user message, a ticket, an email) into a **structured judgment** that directly drives execution. That's powerful, but it creates a new risk the authors call **Jev Program Decision Hijacking (JPDH)**: an attacker who can influence the *context* (any lower-trust text the program feeds the model) can steer the model into emitting a **schema-valid but semantically malicious judgment** — no code change, no weight change, no predefined injection channel required. Prior work studied model-level attacks in isolation; nobody had measured how far this reaches into **real, deployed Jev applications** or what breaks downstream.

## Methodology (technical)

**JevProbe** is the first automated framework to evaluate JPDH end-to-end, tracing adversarial influence along a three-stage path:

1. **Reachable context surfaces** — identify every place attacker-controllable text can enter the Jev pipeline.
2. **Program-consumed decisions** — determine which model judgments the application actually acts on (the insight: application logic maps *diverse* judgments to *observable* decisions, which makes program-grounded attack evaluation possible).
3. **Downstream security consequences** — trace what the flipped decision actually does (e.g., grants access, routes funds, approves a request).

The authors also built **JevProbeBench** to support systematic, reproducible evaluation of JPDH in real applications. Code and dataset are public (GitHub: `xiaoxiaohg8d/JevProbe`).

## Key Results (with numbers)

- Evaluated **228 popular real-world Jev applications** with **2,273 test cases**.
- **81.5%** of applications exhibit **decision flips** under attacker-influenced context.
- Of those flips, **98.6%** lead to a potential **downstream security impact**.
- Compared with the strongest adapted baseline, JevProbe improves the macro-average **Decision Flip Rate by 50.4 percentage points**.
- Existing defenses offer **limited protection** against JPDH.

## What's Novel

- Names and formalizes a new vulnerability class — **Jev Program Decision Hijacking** — distinct from classic prompt injection because the payload must produce a *schema-valid* judgment the program will accept, so simple validation filters don't help.
- First **automated, program-grounded** evaluation that follows adversarial influence all the way from context surface → program decision → security consequence, rather than stopping at the model.
- Provides a **benchmark (JevProbeBench)** and responsible disclosure to maintainers.

## My Connection (to Manny's work)

This is the "LLM-as-control-plane" attack surface — applications where a model's structured output *is* the business logic. For agent red-teaming it means: when an agent's decision is consumed by a program (a router, an approval gate, an access-control check), the exploit isn't a jailbreak — it's a **valid-looking judgment that the program trusts**. JevProbe's three-stage tracing (surface → decision → consequence) is a reusable methodology Manny can apply to any agent whose outputs drive execution, and the 81.5%/98.6% numbers make a compelling case for why input trust boundaries must be enforced in code, not in the model.

## What I Learned (plain English)

When you let a model's structured output make decisions inside a program, an attacker doesn't need to break the model — they just need to nudge its judgment into a form the program will accept. The output looks perfectly valid; the harm is in what the program then *does* with it. Validation of the *format* is not validation of the *intent*. If a model's judgment drives security-relevant logic, you must constrain the reachable context and re-check the decision at the point of consequence.
