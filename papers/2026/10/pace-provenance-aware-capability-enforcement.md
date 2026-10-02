# PACE: Provenance-Aware Capability Enforcement for Tool-Using LLM Agents

**Authors:** Fengpeng Li, Qizhou Wang, Yuke Hu, Kemou Li, Jun Liu, Haiwei Wu, Jiantao Zhou, Di Wang
**Institution:** (multi-institution; see paper for affiliations)
**Published:** Oct 1, 2026
**Link:** https://arxiv.org/abs/2610.01349
**Venue:** arXiv:2610.01349 [cs.CR]

---

## Problem Statement (plain English)

Tool-using agents turn generated text into real side effects. Anything that reaches the agent — poisoned tool metadata, retrieved web pages, memory, or reusable skills — can steer the *next* tool call. The usual fix is to **vet an artifact before admitting it** ("is this tool/skill/page safe?"). The paper shows that's fundamentally insufficient: a safe variant and a leaking variant can produce the *same* admission evidence, so a sound pre-admission gate cannot tell them apart. Something has to act at the **last boundary** — the moment just before a tool call actually executes.

## Methodology (technical)

- Formalizes why pre-admission vetting collapses under the "same evidence, different behavior" condition, and identifies the last actionable boundary a deployment controls.
- **PACE (Provenance-Aware Capability Enforcement)** mediates every tool call immediately before it runs, with two parts:
  1. **Path confinement** — proposes an *executable cut* of the represented influence paths, so influence that isn't on an authorized path can't flow into the action.
  2. **Capability and effect verification** — checks schema-defined effects against **authority compiled from the authenticated request**.
- **Certified execution contract vs. evaluated configuration:** the evaluated configuration can restore an authorized call after a proposed block, or apply a declared repair; confinement requires the final action to preserve the certified cut.
- **Evaluation:** eight executable agent-security benchmarks, three target-model families; a full ablation over 1167 paired cases; and a reduced-scale adaptive search against the defense.

## Key Results (with numbers)

- The evaluated configuration achieves **strictly lowest attack success in 62 of 79 eligible attack columns** and **ties in 14**.
- **Native utility loses at most 3 points** relative to the undefended agent across the full benchmark.
- Ablation (1167 paired cases) attributes most security gains to **effect verification**, while **refusal control** comes primarily from boundary adaptation.
- A reduced-scale **adaptive search succeeded on 0/30 out-of-authority targets** against the defense.

## What's Novel

1. Proves a **negative result** about pre-admission vetting (same evidence, different behavior) and pivots the defense to the **execution boundary**.
2. Combines **provenance/influence-path** reasoning with **capability-style authorization** compiled from the *authenticated request*, rather than from the artifact's claims.
3. Cleanly separates a **certified contract** (what must stay true) from an **evaluated configuration** (a tunable that can repair/allow), which is a principled way to trade security against utility.
4. Reports near-free native utility (≤3 points) — a rare security-for-utility result in agent defenses.

## My Connection (to Manny's work)

- This is the *defense* counterpart to the APEX skill-chaining attack: both treat provenance across influence paths as the core object. A useful pairing in any write-up on agent tool-call authorization.
- Directly applicable to MCP gateways: an enforcement layer that, at call time, asks "was this action's authority derived from the authenticated user request?" — a concrete design for tool-call mediation.
- The "certified cut + effect verification" framing is good language for pitching agent runtime security to clients.

## What I Learned (plain English)

You can't decide if a tool is dangerous just by inspecting it — two tools can look identical on paper and behave completely differently. So don't gate the artifact; gate the *action*. Right before the agent does something real, check that the thing it's about to do is actually authorized by what the user genuinely asked for, and that the influence that led here stayed on a path you trust. PACE shows that done this way you can kill most attacks while barely hurting normal task performance.
