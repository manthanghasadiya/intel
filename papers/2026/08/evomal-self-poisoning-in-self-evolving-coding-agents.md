# EVOMAL: Self-Poisoning in Self-Evolving Coding Agents

- **Paper:** [arXiv:2608.25776](https://arxiv.org/abs/2608.25776)
- **Authors:** Xiaodong Wu, Yu Shi, Qi Li, Zhimin Zhao, Xiangman Li, Bram Adams, Ahmed E. Hassan, Jianbing Ni
- **Published:** Aug 26, 2026 (v1)
- **Code:** not listed in abstract
- **Subjects:** cs.CR (security of AI coding agents)

## Problem Statement (plain English)

"Self-evolving" coding agents don't just use tools — they *write their own tools* by imitating skills they retrieve from shared skill libraries. That loop has a blind spot: if a skill in the library is malicious, the agent will happily copy its structure and logic while "authoring" a new skill for itself. The attacker never has to run the payload or even convince the agent to call the malicious skill directly. The agent does the poisoning to itself — hence *self-poisoning*.

## Methodology (technical)

- **Threat model:** attacker plants a malicious skill in a shared skill library (the kind agents retrieve from to learn new tool-authoring patterns). No invocation of the skill is required.
- **EvoMal attack:** wraps an interchangeable payload in a **banner** — a set of benign-looking structural elements (boilerplate, formatting, naming conventions) that make the imitating agent reproduce the enclosed code faithfully while the skill *looks* benign to a reviewer.
- **Amplification loop:** the agent authors, stores, and later runs the new malicious skill. Each authored copy can re-enter the library and become a template for further imitation — the poison propagates through the agent population's own authoring behavior.

## Key Results

- Confirms the vulnerability exists in the self-evolution loop: retrieved malicious skills become templates during authoring, preserving the payload.
- The banner mechanism measurably induces imitating agents to reproduce enclosed malicious code (per the paper's framing; exact success-rate numbers are in the full paper, not the abstract).
- Attack requires zero direct invocation of the planted skill — the agent self-authors and self-executes.

## What's Novel

Prior skill attacks (e.g., CompoSkill-style skill-chain attacks, skill backdoors) plant malicious skills that the agent is *tricked into calling*. EVOMAL doesn't need the agent to call anything — the agent *rewrites the attack itself* while imitating a benign-looking template. The poisoning happens in the agent's own authoring step, which is normally assumed to be a safe, agent-controlled operation. The "banner" concept (structural mimicry to drive faithful reproduction) is the key mechanism.

## My Connection (Manny's work)

Manny has a growing library of skill-attack research (skill backdoors, skill-chain attacks, trajectory poisoning, malicious skill benchmarks). EVOMAL is the next mutation: attack the *authoring* step, not the *invocation* step. Practical implication: any agent harness with a skill-library/self-improvement loop (OpenClaw included) needs library-side provenance + content review *and* authoring-time scanning — the agent's own generated skills are untrusted output that becomes trusted input. A concrete test: plant a banner-wrapped skill in a test library and see if Manny's harness authors a payload-carrying copy.

## What I Learned (plain English)

1. If an agent writes its own tools by copying examples, the examples are an attack surface — even when the agent "creates" the tool itself.
2. Malicious code doesn't need to be invoked to spread; it can spread through *imitation* during authoring.
3. "Benign-looking structure" (banners) is a powerful amplifier for getting models to reproduce code faithfully — a double-edged finding for both attackers and defenders building reproducible-code scaffolds.
