# From Prompt Injection to Web Exploitation: Revisiting Classic Vulnerabilities in LLM-Integrated Applications

**Paper:** [arXiv:2608.10281](https://arxiv.org/abs/2608.10281) — "From Prompt Injection to Web Exploitation: Revisiting Classic Vulnerabilities in LLM-Integrated Applications"

**Authors:** Spiros Tsigkopoulos, Christoforos Ntantogian
**Institution:** University of Piraeus, Greece
**Published:** 2026-08-10 (announced 08-12, cs.CR)
**Code:** Not listed on abstract page

---

## Problem Statement (plain English)

LLM-powered web apps let user input flow into backend actions — database queries, HTTP requests, file operations, template rendering, API calls. Prompt injection research usually stops at "the model was manipulated," but the real damage happens when that manipulated output reaches a *classic* web vulnerability sink (SQL, XSS, SSTI, command injection, IDOR, CSRF, XXE, SSRF). The paper systematizes this "LLM-mediated web attack" class: attacker-controlled input → LLM transforms it → traditional sink gets exploited.

## Methodology (technical)

- **Taxonomy:** defines LLM2X variants — LLM2SQLi, LLM2XSS, LLM2SSTI, LLM2CommandInjection, LLM2IDOR, LLM2CSRF, LLM2XXE, LLM2SSRF — mapping how model-mediated input reaches each sink type.
- **Core analysis:** the LLM usually does *not* create the underlying vulnerability; it acts as a **mediation layer** — and in tool-enabled settings, as a **confused deputy**, carrying attacker influence into components that trust model-generated content.
- **Experimental case study:** TicketOracle, a Flask-based LLM-integrated web app, evaluates **LLM2SSRF** across five attack scenarios and seven LLMs, measuring susceptibility variation per model.
- **Mitigations** mapped across four layers: prompt, model, application, network.

## Key Results

- Attack surface systematized into eight LLM2X sink classes.
- Confirmed: exploitation depends on **both** insecure application architecture *and* model-specific behavior — susceptibility varies substantially across the seven tested LLMs.
- LLM-as-confused-deputy is the recurring pattern: components trust model-generated content without re-validating it.

## What's Novel

Bridges the gap between two research communities: prompt injection (LLM-centric, stops at the model) and web exploitation (sink-centric, assumes direct user input). It re-frames classic web vulns through the LLM mediation layer and gives pentesters a concrete sink-mapping taxonomy for LLM-integrated apps — plus a reproducible test app (TicketOracle) for LLM2SSRF evaluation.

## My Connection (to Manny's work)

This is a direct methodology upgrade for Manny's engagements: when testing any LLM-integrated app, don't stop at "injection worked" — trace the manipulated output to web sinks (SQLi, SSRF, SSTI, IDOR) using the LLM2X mapping. It's the missing bridge between agent/prompt-injection testing and traditional web pentesting, and the confused-deputy framing is exactly the tool-authorization failure mode that shows up in MCP and agent tooling. The model-variation finding also supports testing the same app across multiple models.

## What I Learned (plain English)

Prompt injection is only the first half of an attack — the second half is whether the app lets the model's tainted output reach a normal web vulnerability. The model isn't creating the bug; it's a middleman that smuggles attacker influence past trust boundaries. When you red-team an AI app, you should test it like a web app *and* an LLM app at the same time, and assume anything the model touches downstream is attacker-influenceable until proven otherwise.
