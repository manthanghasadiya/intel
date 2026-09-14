# SoK: Rethinking Jailbreaking in the Era of Agentic AI: Attacks, Defenses, and Practical Considerations

**Authors:** Md Jueal Mia, Yanzhao Wu, Selcuk Uluagac, M. Hadi Amini
**Institution:** (per arXiv listing — authors affiliated with US institutions incl. Florida International University)
**Published:** arXiv:2609.12413 — submitted 11 Sep 2026
**Link:** https://arxiv.org/abs/2609.12413
**Code:** Not specified in the abstract (controlled empirical study within a common agentic framework)

---

## Problem Statement (plain English)

Most of what we know about "jailbreaking" LLMs was learned on older, weakly-aligned models: craft a prompt, defeat the safety filter, get the harmful answer. But modern models are natively better aligned, *and* they no longer just answer questions — they plan, use tools, keep memory, and talk to other agents. So a huge chunk of the jailbreak literature may be answering a question that has changed. The paper asks: **which established jailbreak-security findings still hold in the agentic era?**

## Methodology (technical)

A **Systematization of Knowledge (SoK)** that rebuilds jailbreak security around the **full agentic execution pipeline**, with unified taxonomies of attacks and defenses across five axes: **user interaction, planning/reasoning, memory, tool use, and inter-agent communication**. It introduces a **security–utility–efficiency evaluation framework** that separates three things commonly conflated: **native harmful-prompt safety**, **adversarial jailbreak robustness**, and **agent-level security outcomes**. Then it runs a **controlled empirical study** of representative attacks and defenses inside a **common agentic framework** so comparisons are apples-to-apples. The paper deliberately measures over-refusal, utility, and latency as first-class costs, not afterthoughts.

## Key Results (with numbers)

Three gaps (qualitative — the abstract reports findings, not a benchmark scorecard):

1. **Strong native alignment does not imply robustness to adversarial jailbreaks.** Better default safety ≠ harder to break under attack.
2. **Defense effectiveness is highly model-, attack-, and component-dependent**, and comes at **substantial cost in over-refusal, utility, and latency** — i.e. defenses often "work" by breaking usefulness.
3. **Low final-response attack success can mask severe intermediate compromise** — planning, memory, and tool interactions may remain unsafe **even when the final response is successfully filtered**.

The conclusion: shift from **response-centric** jailbreak defense to **cross-layer, execution-aware security** that protects **agent state, component transitions, and external actions** while preserving utility and efficiency.

## What's Novel

Most jailbreak work is attack-vs-defense on the *output*. This is the first systematic reframing around the **agentic pipeline as the unit of analysis**, with an evaluation framework that refuses to collapse safety, robustness, and agent-level security into one number. Its most quotable finding — that a *clean final answer can hide a poisoned plan, memory, or tool call* — is a direct indictment of output-filter-based guardrails.

## My Connection (to Manny's work)

This is the conceptual backbone for agent red-teaming. It gives (a) a **taxonomy to structure an attack playbook** (per-layer: interaction, planning, memory, tools, inter-agent), (b) an **evaluation lens that measures intermediate compromise**, not just final-answer refusal, and (c) the argument that **filtering the output is not containment**. For content and demos it supplies the strongest single line: *final-response attack success is a vanity metric — check the plan, the memory, and the tool call.* Directly supports positioning execution-aware control (the Scan-the-Skill gate, NovaFabric evidence) as the real defense.

## What I Learned (plain English)

Catching the bad answer is not the same as stopping the bad action. A modern model can be perfectly aligned at the surface and still have been steered — mid-run, through its memory or a tool response — into doing something harmful, while the final sentence it says to you looks fine. So defenses that only read the output are measuring the wrong layer; the places that matter are the *intermediate* ones: what the agent planned, what it wrote into memory, and what it actually called. Also sobering: many defenses "work" only by making the agent refuse too much and run slower — a defense that destroys the product isn't a fix.
