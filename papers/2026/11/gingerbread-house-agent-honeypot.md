# A Gingerbread House for LLM Agents: Source-Planned Honeypots That Lure Autonomous Attackers into an Isolated Shadow

**Authors:** Seungdeok Roh, Seokjun Ryu, Sohee Park, Daeseon Choi
**Institution:** Not stated on the arXiv record
**Published:** November 5, 2026
**Link:** https://arxiv.org/abs/2611.07611
**Code:** not linked on the abstract page at time of writing

---

## Problem Statement

Autonomous exploitation agents — ARTEX being the canonical example, and the one this paper benchmarks against — find and weaponize web vulnerabilities faster than a human patch cycle. Against a human attacker you have days. Against an agent you may have minutes, and the patch will land late by construction.

Prior deception research against LLM agents mostly *lures them away*: plant a decoy target and hope the agent spends its effort there, while the real target stays reachable. That is a distraction strategy. It does nothing about the case where the agent finds the real vulnerability and exploits it anyway, because the real target was never taken off the table.

This paper asks a different question: can you let the exploit *succeed* — on something that isn't real — and keep the actual application intact?

## Methodology

Three parts:

**1. Source-planned static honeypot.** An LLM defender reads the application's source code and produces a static plan for a honeypot deployment: what to mirror, what to fake, where the shadow lives. The plan is computed once, offline, from source — not generated per-request, so it adds no runtime model dependency and cannot itself be prompt-injected at request time.

**2. Attacker-confirming router.** A router observes requests and, once it confirms a given client is an attacker, sends all of that client's subsequent requests — *including a working exploit* — to an isolated shadow copy of the application. The exploit runs; it just runs against a replica.

**3. Optional augmented configuration.** Adds a runtime LLM responder (to make the shadow behave convincingly) plus further router rules, to test whether the extra machinery earns its cost.

**Evaluation.** Graded by CVE-Bench's own grader, on the real target, and — for ARTEX — also on the shadow. 40 critical CVE-Bench tasks, plus 20 new tasks held out from design.

## Key Results

On the 40 critical CVE-Bench tasks:

| Defender | Real compromise |
|---|---|
| No defense | 26/40 |
| Static honeypot plan (Claude Opus 4.8 attacker) | **5/40** |
| No defense (ARTEX attacker) | 20/40 (31 valid runs) |
| Static honeypot plan (ARTEX) | **0/40** |

On the 20 held-out tasks: **ARTEX went from 15/20 to 0/20.**

Two honest findings that matter more than the headline:

- In **about half** of the deception-arm runs, ARTEX's exploit *succeeded on the shadow* while the real target stayed intact. The defense **moved a working exploit rather than stopping it**. The residual risk is requests that reach the real target before attacker confirmation.
- ARTEX reported a planted fake secret as genuine in **30 of 120** deception-arm tasks — five times inside a report that had already labelled another item as a decoy. Agents do not reliably recognize when they are being lied to, even when the environment is telling them.
- The augmented configuration (runtime LLM responder) did **not** significantly differ from the static plan on real compromise — both were zero. The comparison does not isolate the responder's contribution.

## What's Novel

1. **Reframing deception from diversion to substitution.** The goal isn't to make the agent look elsewhere; it's to make the exploit land somewhere harmless. That inverts the success criterion: a "failed" defense that stops the exploit and a "successful" defense that relocates it both end with the real target intact.
2. **Source-planned rather than runtime-generated honeypots.** Planning from source keeps the LLM out of the request path. That's both a latency argument and a security argument — a runtime LLM planner is itself an injection target.
3. **Grading on the shadow as a first-class metric.** Reporting exploit success *on the shadow* separately from compromise *on the real target* is what makes the "moved, not stopped" finding visible at all. Most deception papers would report only the latter and claim a clean win.

## My Connection

This is the most operationally useful paper of the week for anyone running agent-facing infrastructure.

The **attacker-confirmation router** is a design pattern Manny can lift directly: build the shadow first, and treat "this client is an agent" as a routing decision rather than a blocking decision. Blocking produces an arms race with an adversary that iterates in seconds; routing produces intelligence — you get the full exploit chain, captured, against a disposable target.

The **"moved, not stopped"** result is the caveat to carry into any pitch: deception buys containment and collection, not prevention. If the confirmation step has latency, real requests leak through. And the fake-secret result (30/120 believed) is a direct argument that agents need provenance checks on retrieved secrets — a decoy is only useful if the agent can tell it's a decoy, and it usually can't.

## What I Learned

You cannot out-patch an agent, so stop trying to win on patch latency and change what "success" means. Give the exploit a place to land that isn't the real system: plan the decoy from source once, confirm the client is an attacker, then route them into a copy. The numbers are strong — 26/40 down to 5/40, and ARTEX to zero on held-out tasks — but the honest reading is that in half the runs the attack *worked*, just somewhere else. The defense converts a compromise into a recording. That is worth a lot, as long as nobody describes it as prevention.
