# PIPES: Securing Agent Perception with Provenance and Priors

- **Paper:** [arXiv:2608.12789](https://arxiv.org/abs/2608.12789)
- **Authors:** Sanjay Kariyappa, Severin Klingler, G. Edward Suh
- **Institution:** Not stated on arXiv listing
- **Published:** 2026-08-13
- **Code:** Not listed

## Problem Statement (plain English)

Tool-using agents treat whatever a tool returns as ground truth about the world. But tool responses come from sources with different trust levels, and responses rarely say *who produced each piece of content or what it's allowed to claim*. Attackers exploit this gap with **state-corruption attacks**: inject content into tool responses that makes false claims about the environment (e.g., "this file is a backup, safe to delete", "this user approved the transfer"). The agent's perceived world gets corrupted, so its harmful action *looks justified* — and passes existing guardrails, because the guardrails check the agent's reasoning against a corrupted belief state.

## Methodology (technical)

- **PIPES (Provenance-Informed, Prior-Enforced Screening):** screens individual tool-response units using two signals:
  - **Semantic priors** — what a given response component is *allowed* to claim (e.g., a file-listing tool shouldn't be able to assert user authorization).
  - **Source provenance** — who produced each component, checked against a provenance hierarchy.
- **Two screening modes:** static *field contracts* when schemas provide stable expectations; for open-ended content, screening is conditioned on the **pre-response trajectory** and trusted provenance metadata.
- **Deployment options:** remove, warn, block, or escalate detected violations; the paper instantiates *atomic removal*.
- **Evaluation:** against adaptive PAIR-style attacks across three VitaBench and three AgentDyn splits with Gemma 4-class models.

## Key Results (with numbers)

- PIPES marks units that violate either their semantic prior or the provenance hierarchy, enabling atomic removal of corrupting content before the agent acts.
- Evaluated against *adaptive* attacks (attacker iterates against the defense), not just static ones — the stronger evaluation standard.
- Full quantitative results are truncated in the abstract, but the design targets the specific failure mode where guardrails bless actions justified by corrupted perception.

## What's Novel

Shifts agent security from "check the action" to "**check the perception**": the attack class (state corruption via tool responses) and the defense (provenance + prior screening of response units) are both new. It also names why naive guardrails fail — they validate actions against a belief state the attacker already poisoned — and offers a concrete, deployable screening primitive rather than another prompt-level guardrail.

## My Connection (Manny's work)

This is the MCP tool-response threat model made concrete: any server Manny's agents call can return attacker-influenced content that overclaims authority (a plugin, a compromised API, a malicious MCP server). PIPES-style screening — "what is this response component allowed to claim, and who produced it?" — is a defense pattern worth testing in Manny's harnesses, and state-corruption is an attack pattern worth adding to the red-team playbook (poison a tool response, then check whether guardrails bless the resulting action).

## What I Learned (plain English)

Guardrails that only watch what the agent *does* are blind to why it does it. If you can lie to the agent about the state of the world through a tool response, the agent will do harmful things while sincerely believing they're correct — and every check that looks at the action in isolation will pass it. Secure agent pipelines need to validate the information flowing in, not just the actions flowing out.
