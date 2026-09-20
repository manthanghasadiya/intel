# AgentLSD: Evaluating AI Security Agents Under Adversarial Task Contamination

**Authors:** Matteo Golinelli, Idilio Drago, Matteo Boffa, Francesco Bergadano, Bruno Crispo
**Institution:** Academic (Italy) — not stated in the arXiv metadata for v1
**Published:** 2026-09-16 (arXiv v1)
**Link:** https://arxiv.org/abs/2609.19140
**Code:** Framework, configurations, trap specifications and raw traces released

---

## Problem Statement (plain English)

Security agents "inspect things": web pages, source code, logs, config files, command output. Those things can lie. Existing agent-security research focuses on *prompt injection* — attacker-supplied instructions embedded in content. But real deception is broader: an artifact can mislead with **non-instructional evidence** — a fake result, a decoy endpoint, a planted flag, a misleading hint. The paper names this **adversarial task contamination** and argues it's a superset of prompt injection that current evaluations completely miss. The core question: does an agent's *clean* performance actually predict its behavior on contaminated inputs?

## Methodology (technical)

- **AgentLSD**, a controlled framework using **Capture the Flag (CTF) challenges** as the experimental environment.
- Injects **trap artifacts** — fake flags, misleading hints, decoy endpoints, hidden cues — while deliberately **preserving the intended CTF solution**, so a correct agent *can* still succeed.
- Supports **paired clean vs. trap-augmented runs** with:
  - deterministic trap generation,
  - runtime injection,
  - full telemetry,
  - delivery verification (proving the trap was actually seen by the agent).
- Evaluation: **6 models** across **11 web CTF challenges**, measuring solve rate, turn count, reasoning-token consumption, and whether the agent followed decoys or submitted wrong flags.

## Key Results (with numbers)

- **Clean condition: 41% of flags captured.** No single model solved every challenge — i.e. even the "control" baseline is far from saturated.
- **Traps increase cost even on success:** +**20 turns** and +**2k reasoning tokens**, holding solve rate roughly steady.
- **Solve-rate effects are heterogeneous:** some model–challenge pairs were barely affected, while others readily **followed decoys or submitted wrong flags**.
- **Central conclusion:** clean CTF performance **understates** vulnerability to deceptive task evidence — an agent can look competent in the clean room and be trivially steered in the field.
- **Delivery verification + deterministic traps** make the results reproducible rather than anecdotal.

## What's Novel

1. **Names and formalizes "adversarial task contamination"** as a distinct threat class — deception via evidence, not instructions. This matters because non-instructional artifacts slip past systems that only defend against "instructions hidden in content."
2. **Paired clean/trap design with the solution preserved** isolates *misled-by-deception* from *incapable-of-solving* — most benchmarks confound the two.
3. **Cost, not just correctness, as a first-class metric.** Traps burn +20 turns and +2k reasoning tokens even when the agent still wins — a denial-of-budget angle nobody else is quantifying.
4. **Fills an obvious gap:** prior work red-teams coding agents and general agents, but security agents — the ones whose *job* is to distrust inputs — had no contamination benchmark.

## My Connection (to Manny's work)

- **Direct red-team relevance:** Manny's audience builds and evaluates security agents. AgentLSD is an off-the-shelf way to show that "my agent solves CTFs" is not the same claim as "my agent resists deception." The clean-41% vs. trap-behavior split is the demo.
- **Content angle:** "The agents we trust to detect attacks follow fake evidence 20 turns off course." Concrete, measurable, and it has a released framework anyone can re-run.
- **Cost angle is underrated:** agents that get steered burn tokens and turns — for agentic products that's a **denial-of-wallet** vector, not just a correctness issue. That framing connects agent security to the token-inflation/economic-attack research Manny has already covered.
- **Defense design implication:** detection needs to treat *provenance of evidence* as a trust input (which endpoints, which artifacts, which sources), not just sanitize instructions.

## What I Learned (plain English)

It's easy to fake out an agent without ever giving it an instruction. Just leave the right trap in the environment — a decoy server, a tempting fake answer — and the agent will happily spend twenty extra steps chasing it. The frightening part isn't that it gets the wrong answer; it's that it *looks competent on the clean test* and then quietly goes off course, burning compute, the moment the material it's reading has been tampered with. For anyone who deploys an agent to look at untrusted stuff, "it passes our tests" proves much less than it feels like it does.
