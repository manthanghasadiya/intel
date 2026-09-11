# BlueSTAR: Tiered Agentic Architecture for Autonomous Cyber Defense

**Authors:** Simona Boboila, Xavier Cadet, Edward Koh, Daniel Balasubramanian, Dirk Van Bruggen, Peter Chin, Alina Oprea
**Institution:** (not stated in the arXiv listing; authorship includes well-known academic and industry security researchers)
**Published:** September 10, 2026
**Link:** https://arxiv.org/abs/2609.11852
**Code:** (none listed at time of writing)

---

## Problem Statement

Attacks are increasingly automated, collapsing the time human analysts have to detect, reason about, and respond to intrusions. LLMs look like a natural foundation for autonomous defense because they can correlate heterogeneous evidence and reason about unseen threats — but you cannot simply point one at operational security telemetry. Raw logs arrive faster than models can process, individual events are ambiguous on their own, and *unconstrained* LLM actions can themselves cause serious operational damage. The paper asks: how do you get LLM reasoning into the SOC without either drowning the model in logs or handing it unsafe authority?

## Methodology

BlueSTAR is a **tiered agentic architecture** for autonomous cyber defense in enterprise IT/OT networks. Rather than feeding raw telemetry to an LLM, it first **transforms high-volume security telemetry into compact indicators of compromise (IOCs)**, then performs agentic reasoning and response over that reduced representation. The tiering is what lets fast, deterministic handling cover known threats while slower, LLM-driven reasoning handles ambiguous or novel situations.

The authors also introduce a **resilience metric** that jointly captures three things: attacker reach, impact on mission-critical assets, and the disruption caused by the *defensive* actions themselves — i.e., a defense that locks everything down is penalized for the outage it causes.

Evaluation was run on **two live enterprise IT/OT cyber ranges** against **seven attack chains** based on real-world intrusion techniques.

## Key Results

- BlueSTAR **retains the fast containment of deterministic response** for known threats.
- It **successfully defends against attacks that require contextual and cross-cycle reasoning**, including credential theft, repeated compromise, concurrent attackers, and attacks against physical processes (OT).
- The resilience metric gives a single number that trades off defender-induced disruption against attacker impact — useful for comparing response policies rather than just detection rates.

## What's Novel

- **Telemetry-to-IOC compression as the front of an agentic pipeline** — solving the "logs outpace the model" problem by changing the model's input rather than demanding a faster model.
- A **tiered** architecture that deliberately blends deterministic response with LLM reasoning instead of replacing one with the other.
- A **resilience metric** that counts the cost of defensive action, which most detection/response evaluations ignore.

## My Connection

This is the defender-side mirror of the agentic-offense work dominating the news cycle — the PaperCut "hundreds of AI agents" campaign and Anthropic's threat report (Sept 10) both describe adversaries orchestrating agents; BlueSTAR describes doing it on defense with explicit attention to not letting the agent act unilaterally. For red-team work it doubles as an evaluation target: the resilience metric and the "attacks against physical processes" chains are exactly the scenarios to probe for agent over-action (a defender agent that over-responds is a denial-of-service primitive). It also pairs with EBL-Core — both insist that an agent's authority to act must be bounded and justified.

## What I Learned

The hard part of autonomous cyber defense isn't making the model smart enough — it's deciding what to even show it (raw logs are too much and too ambiguous) and then constraining what it's allowed to *do* once it decides. And a good defense is measured not just by what it stops but by how much disruption it causes while stopping it.
