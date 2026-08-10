# HarnessSafe: Evaluating Safety Across Persistent Carriers in Agent Harnesses

- **Authors:** Xiao Zhang, Yusheng Wang, Yuhao Fei, Dongyuan Li, Zian Liang, Liuyu Xiang, Hongxun Gu, Zhaofeng He
- **Institution:** Not listed on arXiv abstract page (last author Zhaofeng He's group is affiliated with Beijing University of Posts and Telecommunications)
- **Published:** Submitted Aug 7, 2026; announced Aug 10, 2026
- **Link:** https://arxiv.org/abs/2608.06984
- **Code:** Not stated in abstract

## Problem Statement (plain English)

Modern agent harnesses (the scaffolding around the model: memory files, skill directories, tool definitions, shared artifacts) persist state *across* tasks and sessions. That persistence is a delayed bomb: attacker-influenced content can slip in during one session, sit quietly in a memory or skill file, cross system boundaries, and later corrupt the execution of a completely benign request. Existing safety benchmarks mostly test one carrier or one harness at a time, and their single "attack success rate" number doesn't tell you *how* the attack propagated or where defenses actually stopped it.

## Methodology (technical)

- **Benchmark:** HarnessSafe — 328 executable cases across **seven persistent-carrier families**, evaluated on most mainstream agent harnesses.
- **Persistent-Risk Lifecycle:** each case is specified as a lifecycle tracing attacker influence through four phases: (1) initial entry of attacker-controlled content, (2) persistence across carriers and system boundaries, (3) a later benign trigger, and (4) an observable violation.
- **Multi-stage, trace-based evaluation:** uses observable execution evidence to determine how far each attack chain progresses and at which stage it is stopped — rather than a single binary success/fail.

## Key Results (with numbers)

- Containment is **carrier-specific**: some carriers propagate risk readily, others don't — and this depends heavily on the specific harness-model configuration.
- Both the **harness and the model backend** substantially shape containment outcomes.
- End-to-end attack success rates **cannot reflect distinct lifecycle progression patterns** — two attacks with the same success rate can fail (or propagate) at completely different stages.

## What's Novel

- First benchmark to systematically cover **seven persistent-carrier families** at once, on mainstream harnesses.
- Replaces binary attack-success metrics with **lifecycle-stage tracing**: the field's first attempt to measure *where* attacks are stopped, not just *whether* they succeed.
- Makes the case that the harness is a first-class security control, not just the model.

## My Connection (to Manny's work)

This is the audit methodology for the "agent persistence attack surface" Manny keeps hitting in the wild (skill files, memory, MCP tool state). The lifecycle-tracing approach is directly usable as a checklist when reviewing agent deployments: for each carrier (memory, skills, tools, shared artifacts), trace entry → persistence → trigger → violation and document where containment engages. Also pairs naturally with the Claude Code auto-mode change: classifier decisions are per-call, but HarnessSafe shows risk lives across calls in persistent state.

## What I Learned (plain English)

Where an attack is stopped matters more than whether it succeeded — and "success rate" numbers lie about safety. When auditing an agent system, the question isn't just "can this be attacked?" but "which carrier did it travel through, and which layer finally caught it?" A harness with good persistence hygiene can be safer than a smarter model in a sloppy harness.
