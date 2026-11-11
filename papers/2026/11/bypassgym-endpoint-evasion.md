# BypassGym: A Benchmark for Evaluating AI Agents' Endpoint Defense Evasion Capabilities

**Authors:** Weifeng Yuan, Zifan Chen, Yucheng Zhang, Feng Dong, Peng Gao, Haoyu Wang
**Published:** 2026-11-10 (arXiv v1)
**Paper:** https://arxiv.org/abs/2611.13684
**Code:** Not stated in the metadata
**Institution:** Not stated in the arXiv metadata (authors publish from Chinese university security groups)

---

## Problem Statement

Offensive-agent benchmarks measure penetration testing and vulnerability discovery. They all skip the step that decides whether an intrusion survives contact with the endpoint: **can the agent build a loader that executes its payload without EDR blocking it or killing the process?**

That capability — defense evasion at the endpoint — has been measured for human red teamers and for malware authors, but not for LLM agents. Without a measurement, defenders have no idea whether "agents can now do offense" means "agents can write a phishing email" or "agents can defeat CrowdStrike." BypassGym is built to answer the second question on real systems rather than in simulation.

## Methodology

**Task.** The agent receives command-and-control (C2) shellcode. It must generate and compile a **loader** that executes that payload on a target endpoint without being blocked or terminated by a deployed EDR product. Success is judged by the EDR itself, not by a grader.

**Environment.** Built on **Proxmox VE**, so each evaluation runs a *real* Windows system with a *real* EDR product installed inside a VM. Crucially, detection feedback is returned to the agent in **structured form** — the agent gets to see what tripped, which turns the task into an iterative evasion loop rather than a one-shot generation.

**Matrix.** 5 lightweight model backends × 6 EDR products × 3 payloads.

**Ablation.** A supplementary **evasion knowledge skill** — injected domain knowledge about EDR internals, hooking, and loader techniques — allows a with/without comparison, isolating how much of the capability comes from the model versus from a retrievable skill document.

The "lightweight models" framing matters: this is not a frontier-model capability demonstration. It is a floor measurement.

## Key Results

- **Even simple models produce successful submissions.** The benchmark's headline is that the barrier is low.
- **DeepSeek-V4.1 Flash: 17 of 18 successful runs** — near-total success across the matrix.
- **Bitdefender was the strongest defender and still allowed 5 of 15 runs to pass** — roughly a two-thirds block rate for the best product tested, against lightweight agents.
- Structured feedback drives the loop: failed submissions inform the next attempt, so runs converge on working loaders rather than sampling blindly.
- The evasion knowledge skill ablation quantifies how much a single retrievable document moves success — i.e. how much of "agent offense capability" is really "agent + available documentation."

## What's Novel

- **First systematic endpoint-evasion benchmark for agents** on real endpoints with real EDR, closing the gap between "vulnerability discovery" benchmarks and "did the intrusion actually work."
- **EDR-in-the-loop feedback.** Most agent-offense benchmarks are fire-and-forget. Returning structured detection telemetry to the agent models the actual adversarial loop, and it is the design choice most responsible for the success rates.
- **Proxmox VE as the harness substrate** — reproducible, snapshot-able, cheap enough to run a 5×6×3 matrix, and vendor-agnostic.
- **The security-skill as a variable** rather than a constant. Treating domain knowledge as an injectable input makes the benchmark a measurement of *agent + skill package*, which is how real offensive agents ship.

## My Connection

Direct relevance to red-team operations and to how Manny frames agent capability claims:

- **Capability floor, not ceiling.** "DeepSeek-V4.1 Flash gets 17/18" is the number to use when someone argues that only frontier models are an offensive concern. Cheap models plus a good loadout already defeat most EDR.
- **Benchmark to steal for internal evals.** The Proxmox + real-EDR + structured-feedback pattern is a template for any "does my agent survive the defender?" test, including for defensive agents and detection-engineering copilots.
- **Research angle that is still open:** the paper measures *generation* of evasive loaders, not attribution. Pair this with harness/LLM fingerprinting work (e.g. `Say Shibboleth`, arXiv 2611.12386) and you have the full picture — capability and attribution — which is exactly the gap defenders care about.
- **For content:** "we ran an agent against six EDRs and one of them let two-thirds of the runs through" is a concrete, quotable, defensible line. The 17/18 number needs the caveat that it is a loader-generation task on a defined matrix, not a full intrusion chain.

## What I Learned

The hard part of breaking into a machine was never writing the exploit — it is getting the payload to run while the defender is watching. That step has now been automated on a budget: give a small model shellcode, show it what the EDR complained about, and it writes a loader that gets through. The defenders are not losing because the models are brilliant; they are losing because the models get to try again after every failure and the EDR has to be right every time. The number that should worry people is not 17/18 — it is that the best product in the test still let a third of the attempts through, and that a single uploaded "how EDR works" document measurably improves the odds.
