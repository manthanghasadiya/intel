# Securing Web Agents with the Compile-Then-Execute Paradigm

**Authors:** Annabella Chow, Jinhao Zhu, Sylvie Venuto, Abdulaziz Alharbi, Basel Alomair, Raluca Popa, David Wagner, Julien Piet — UC Berkeley
**Published:** November 4, 2026
**Link:** https://arxiv.org/abs/2611.06039
**Code:** Not released at time of writing
**Subjects:** cs.CR
**System name:** Baya

---

## Problem Statement

Web agents built on ReAct interleave reasoning and action. At every step the model observes the live page — DOM, accessibility tree, rendered text — and picks the next action. That observation channel is the vulnerability: the page is attacker-influenceable, so the model's **control flow** is a function of untrusted input.

This is why prompt injection against browsers is so hard to defend. It is not that the model fails to recognise an instruction; it is that the architecture gives untrusted text direct authority over which action happens next. A defence that lives inside the model (classifiers, instruction hierarchies, delimiting) is trying to restore a boundary the architecture never had.

The paper's move is to change the architecture so the boundary exists.

## Methodology

**Baya** splits the agent into two phases with a hard wall between them.

**Phase 1 — Compile.** The user's request is translated into an executable **Python program** that calls a **Trusted Website API**. This happens *before the agent touches the live site*. The planning model never observes live website content. The compiled program is the complete plan, and its control flow is fixed at compile time.

**Phase 2 — Execute.** The program runs against the live site. Untrusted page content can only do two things:
- become a **value** passed into a pre-defined operation, or
- **select among branches that were already defined during compilation**.

What it cannot do is redefine the task, or cause the model to synthesise a new website operation at runtime. This is the whole security property, and it is structural rather than probabilistic — no classifier is being asked to win a race.

**The Trusted Website API** is the enabling piece. It gives the compiler a unified, typed view of the site's action space (the operations a user could take), which is what makes ahead-of-time planning possible at all. ReAct needs stepwise discovery precisely because it has no such view; it must poke the site and read back to learn what it can do. With an API, the compiler can plan over the full action space without seeing the site.

**Evaluation.**
- Utility: **581 valid WebArena Verified tasks**
- Security: **WASP**, a web-agent security benchmark, measuring attack success rate under prompt injection

## Key Results

| System | WebArena Verified utility | WASP attack success rate |
|---|---|---|
| Browser-only ReAct | 55.1% | **25.5%** |
| API-enabled ReAct | 66.7% | **16.2%** |
| **Baya (compile-then-execute)** | **77.8%** | **0%** |

The headline is not the 0% — benchmarks always invite the question of what the benchmark missed. The headline is that **security and utility moved in the same direction**. Every prior attempt to harden a ReAct agent against injection has cost task performance; the standard trade in the literature is "we cut ASR from 25% to 4% and lost 12 points of utility." Baya gains 22.7 points of utility over browser-only ReAct while driving ASR to zero.

That inversion is the evidence that the architectural framing is right. Compile-then-execute is not a tax on capability; it is a better way to drive a site, and it happens to also be safe.

## What's Novel

1. **Structural rather than statistical defence.** The guarantee comes from the order of operations (plan before observing), not from a detector. There is no adversarial-input distribution to drift away from.
2. **The Trusted Website API as the enabling primitive.** This is the genuinely load-bearing idea and the one with the hardest adoption path. Without an API exposing the site's action space, you cannot compile ahead of time. The paper is implicitly arguing that agent-safe web interaction requires sites to publish a machine-readable action surface — which is a much bigger claim than a single system.
3. **Compilation as the trust boundary.** Turning a natural-language task into a fixed program is the moment authority is granted. Everything after that is data. This is a clean, auditable model: you can read the program before it runs.
4. **A utility win, not just a security win** — which is the only kind of security result that ships.

## My Connection

This is the strongest architectural answer to indirect prompt injection published this cycle, and it directly contradicts the direction most of the agent-security market is heading. The dominant product category is runtime guardrails: a model or classifier watching the agent's trajectory and blocking bad actions. Baya says the trajectory should never have been attacker-influenceable in the first place.

For Manny's agent work, three things transfer beyond browsers:

- **"Plan then execute" is a general pattern.** Any agent whose plan is derived only from the trusted user request, and whose execution only consumes untrusted data as *values*, is immune to control-flow hijacking by construction. That applies to code agents, shell agents, and MCP orchestrators.
- **Fixed control flow is auditable.** A compiled plan can be diffed, reviewed, and policy-checked before execution. A ReAct trajectory can only be reviewed after the fact, which is too late.
- **The 0% is a design claim, not a benchmark claim.** Worth stating carefully in any writeup — "0% ASR on WASP" is the measurement; "untrusted data cannot alter control flow" is the reason.

The obvious follow-up for a red team: what happens when the *compiler* is fooled — when the user's own request is the injection vector, or when a site's action space is adversarially shaped so the only compilable plan is a harmful one? The paper's boundary is the user request; that boundary is still soft.

## What I Learned

A normal web agent looks at a webpage, decides what to do next, does it, then looks again. Because it's looking at a page an attacker may control, the attacker effectively gets a vote on the agent's next move. That is the whole vulnerability, and no amount of "spot the malicious instruction" training fixes it, because the agent is *supposed* to read the page.

Baya changes the order: figure out the entire plan first, from the user's request alone, and write it down as a program. Then run that program against the site. Now the page can only supply values — a name, a date, a quantity. It can't change what the program does.

The surprising part is that this made the agent *better* at its job, not worse: 77.8% success versus 55.1%. Planning ahead with a proper map of what the site can do beats poking around blindly. Safety here wasn't a cost — it was a side effect of doing the task properly.

The catch: it only works if websites expose a clean list of actions an agent can take. Most don't yet. So the real fight is over whether that becomes a standard.
