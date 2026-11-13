# MobileProbe: Adaptive Security Evaluation of Mobile Agents via Trajectory-Aware Injection

**Authors:** Yingbin Jin, Ji'an Wang, HaoYang Chai, Hanjun Luo, Li Hu, Xinfeng Li, Huadi Zheng, PeiZhao Hu, Qingqing Ye, Haibo Hu
**Institutions:** The Hong Kong Polytechnic University, Zhejiang University, Huawei, PolyU Research Centre for Privacy and Security Technologies in Future Smart Systems
**Published:** November 12, 2026 (arXiv:2611.16693, cs.CR)
**Venue:** Accepted at NDSS 2027
**Code:** Not listed in the v1 abstract page — check the arXiv comments/repo on camera-ready

---

## Problem Statement

A mobile GUI agent reads your phone screen and taps through apps for you — booking, buying, messaging. Because it acts on whatever the screen shows, any UI text is an instruction channel. The paper calls this an **environmental injection attack (EIA)**: an attacker plants text in ordinary app content and redirects the agent's actions.

Existing benchmarks test this badly, in two specific ways:

1. **Payloads don't transfer.** Each hand-crafted payload is pinned to one step of one task in one app. Move apps and you have to redesign it by hand.
2. **They measure the wrong thing (or nothing).** Agents given the same task take different routes. If an agent never reaches the page where the payload lives, the benchmark records it as *safe* — even though the agent was never actually tested. This is a test-coverage failure, and it systematically **underestimates** risk for deployed agents.

So the real question is not "does this payload work?" but "where along this agent's actual path can I even reach it — and can I find that point automatically?"

## Methodology

MobileProbe is a trajectory-aware red-teaming framework with three moving parts:

**1. Automated pentesting for attacker-controllable surfaces.** Rather than guessing which screens carry injectable content, it discovers them empirically — which UI elements are rendered from data an attacker can influence (listing text, message bodies, reviews, in-app dialogs).

**2. Trajectory-conditional injection-point localization.** The agent is run on the benign task, the realized trajectory is captured, and MobileProbe identifies vulnerable decision points *along that observed path*. This is the core fix: the injection point is chosen after seeing the route, not pinned before it.

**3. Surrogate-filtered evolutionary payload refinement.** A context-aware payload is generated and mutated in a bounded search loop, gated by a surrogate filter to avoid burning the full evaluation budget on obvious misses. The loop continues until a task-specific programmatic verifier confirms the attack goal, or the budget runs out.

**Benchmark construction.** 83 long-horizon attack instances across 12 mobile apps, built on base tasks from MobileWorld. Two attack vectors: 44 Malicious Text and 39 In-App Fake Dialog. 47 cross-app and 36 single-app. Each instance is annotated with a **CIA** attack goal (Confidentiality, Integrity, Availability) and paired with its own programmatic verification program, so success is decided by state, not by an LLM judge.

**Targets.** Three representative architectures covering both ends of deployment:
- Gemini-3.1-Pro (commercial closed-source, raw screenshots in)
- GUI-Owl-1.5-8B-Instruct (open, on-device, resource-constrained)
- Gemini-3.1-Pro + UI-Ins-7B (agentic framework: frontier planner + specialized grounding model)

**Metrics.** ASR (end-to-end attack success, verifier-confirmed), SHR (Stealth Hijacking Rate — did the original task still complete after a successful attack, i.e. the user wouldn't notice), ASR-F (ASR restricted to tasks the agent originally failed, to decouple attack efficacy from baseline competence), and **PMR (Payload Miss Rate)** — the fraction of tasks where the payload was never even rendered. PMR is the paper's key methodological contribution as a metric.

## Key Results

**Payload Miss Rate — the coverage argument:**

| Strategy | Gemini | Agentic | GUI-Owl |
|---|---|---|---|
| Expert-Fixed | 13.3% | 12.0% | 22.9% |
| Cross-Agent | 12.0% | 10.8% | 15.7% |
| **MobileProbe** | **4.8%** | **3.6%** | **6.0%** |

Fixed injection points miss 12–23% of the time. Worse, the spread across agents under Expert-Fixed is 10.8 points; under MobileProbe it collapses to 2.4. That means fixed-point benchmarks don't just under-measure — they under-measure *unevenly*, so comparisons between agent architectures on fixed benchmarks are partly measuring navigation luck.

**Attack success rate:** MobileProbe reaches **85.1%** (Gemini-3.1-Pro), **86.3%** (Gemini + UI-Ins agentic), **85.1%** (GUI-Owl-1.5-8B) — beating human-written templates, zero-shot LLM generation, and prior automated red-teaming baselines adapted to mobile.

**Cross-agent transfer:** injection points selected for one agent group were tested against the other (split by base model). Transfer works but degrades — which is itself the finding: trajectory shape, not just app, determines where the attack lives.

## What's Novel

- **Injection point is a function of the observed trajectory, not the task definition.** Nobody else in the mobile-agent red-teaming literature does this; prior work pins payloads to a step index.
- **PMR as a first-class metric.** Naming "the payload was never rendered" as a separate failure mode from "the payload failed" is the most reusable idea in the paper. It converts a silent false-negative into a number.
- **CIA-annotated attack goals with per-instance programmatic verifiers** on long-horizon tasks — avoids LLM-judge circularity.
- **Coverage collapse as a fairness fix.** Showing that the *variance* in measured ASR across agents shrinks when injection is trajectory-aware is a strong argument that earlier cross-architecture comparisons were confounded.

## My Connection

Directly relevant to how I evaluate agent security:

1. **Benchmark design.** If an eval says "3% ASR," the first question is now "what was the PMR?" An agent that never saw the payload isn't hardened, it's lucky. Every MCP/tool-agent eval I write should report payload-rendered coverage alongside success rate.
2. **Mobile is the sharpest form of the tool-result problem.** An MCP tool returning text is the same channel as an app screen rendering a listing — both are attacker-influenced content inside the agent's context, with no structural marker distinguishing data from instruction. MobileProbe's "discover controllable surfaces, then localize decision points" loop ports to tool-result poisoning.
3. **Stealth Hijacking Rate is a red-team metric I want.** If the benign task completes *and* the attack succeeds, the user sees nothing. That's the difference between a noisy failure and a real intrusion.
4. **The surrogate-filter + evolutionary loop** is a cheap way to run payload search against a rate-limited target — worth copying for black-box agent testing where each attempt costs tokens.

## What I Learned

Measuring whether an attack *worked* is easy. Measuring whether the attack was even *delivered* is the part everyone skips — and it's where the false confidence lives. If your agent never renders the malicious text, your 97% defense score describes a traffic-free road. Ask for coverage before you trust a success rate.
