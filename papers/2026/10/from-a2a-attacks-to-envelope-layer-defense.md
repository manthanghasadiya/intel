# From A2A Attacks to Envelope-Layer Defense: Red-Teaming LLM Agents

**Authors:** Yuelin Han
**Date:** Submitted 30 Sep 2026 · Announced 2 Oct 2026
**Link:** https://arxiv.org/abs/2610.00392

---

## Problem Statement

Agent-interaction protocols such as **ACP** and **A2A** have pushed LLM agents toward multi-agent collaboration, and with them a new security reality: **a task sent by a remote peer over A2A is treated as a legitimate request.** That makes inter-agent messaging a natural, protocol-sanctioned channel for **indirect prompt injection**.

Compounding this, existing agent-security evaluations rely on a single blunt metric — **attack success rate (ASR)** — which cannot tell the difference between:

- the LLM *recognizing* malicious content and refusing, versus
- an agent-layer *mechanism* blocking execution.

Both collapse to "attack failed," so defenders learn nothing about *where* the defense actually worked.

## Methodology

Two contributions:

**1. A2A-TIBA** — an attack principle combining indirect prompt injection with bypass circumvention, executed in three stages: **implant → command → exfiltration**. The attacker first induces the target agent to deploy a **callback interaction program**; once that's in place, the attacker can **issue commands bypassing the agent layer entirely** — persistence past the point of injection.

**2. GDA Measurement** — a red-team testbed method using:
- raw context capture via an **LLM gateway**,
- **dual data preservation** (keep both raw and processed),
- **agent-based autonomous judging**.

It replaces the single ASR number with **four attack outcomes, Class A/B/C/D**, decomposed into:
- **semantic refusal rate**,
- **semantic breach rate**,
- **interception rate**,
- **penetration rate**.

The analysis surfaces the **"envelope layer"** — the channel through which malicious content *enters* an agent — as a new defense dimension.

**3. ELA-ITL** — a three-layer isomorphic attack/defense model:
- defense = envelope packaging / LLM recognition / agent interception;
- attack = implant channel / prompt optimization / execution mechanism.

## Key Results

- Tested across **15 agent front-end × LLM back-end combinations**, with a **1,000-case dataset**.
- Verifies attack effectiveness of A2A-TIBA (remote-peer requests as an injection vector + post-injection bypass).
- Confirms the finer-grained GDA Measurement distinguishes *why* defenses succeed/fail where raw ASR cannot.
- **Finding:** adding malicious-prompt *labels* at the envelope-packaging layer (A2A, tool, and memory channels) **significantly improves LLM recognition** of malicious content — i.e. how you frame/annotate the inbound channel materially changes model behavior.

## What's Novel

1. **ASR decomposition** into refusal/breach/interception/penetration — a real diagnostic, not a pass/fail.
2. **Envelope layer** as a named, testable defense surface distinct from the model and the agent harness.
3. **Isomorphic (three-layer) mapping** of attack stages to defense layers, enabling layer-by-layer measurement.
4. **A2A/ACP as an injection transport** — treats protocol trust as the vulnerability, not just the payload.

## My Connection (Manny's Work)

This is a red-team methodology paper — exactly the measurement discipline Manny's agent-security testing needs. The four-outcome metric set is directly adoptable for evaluating MCP/agent defenses, and the "envelope layer" reframing says *label and sanitize the transport, not just the prompt*. It complements the tool-call authorization work (ToolFence/PACE) by covering the **inter-agent transport** they mostly ignore.

## What I Learned (Plain English)

When an agent gets a task from another agent, it assumes the request is honest — that's the hole. And "the attack didn't work" is a useless result; you need to know whether the model caught it, the harness blocked it, or it slipped through. The paper's practical tip is almost free: label the channel a request came in through (peer message, tool output, memory) and the model gets measurably better at spotting bad content.
