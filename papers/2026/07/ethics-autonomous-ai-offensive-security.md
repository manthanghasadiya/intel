# Paper: The Ethics of Autonomous AI Agents for Offensive Security

- **Authors:** Andreas Happe, Jürgen Cito, Jasmin Wachter
- **Institution:** TU Wien (inferred from author affiliations)
- **Published:** July 22, 2026
- **Link:** https://arxiv.org/abs/2607.20255
- **Venue:** FAIEMA 2026 (accepted)
- **Code:** Not applicable

---

## Problem Statement (Plain English)

AI agents for offensive security (automated penetration testing, autonomous red-teaming) are getting powerful quickly. But they break existing ethical frameworks because they're fundamentally different from traditional pen-testing tools in three ways: (1) nobody — not even the developers — can fully explain why the agent took a particular action; (2) the impact is open-ended because the agent might use other AI models or tools you didn't anticipate; and (3) the skill barrier has dropped so much that people who'd never be able to hack manually can now deploy capable offensive agents. Existing dual-use guidelines (for cybersecurity tools and AI models separately) don't handle this combination.

## Methodology (Technical)

This is a theoretical/analytical paper, not an empirical one. The methodology:

1. **Define three dimensions of indeterminacy** in agentic security tools:
   - *Action indeterminacy:* Actions come from a non-deterministic policy (LLM generation), resistant to ex-ante safety review and ex-post explanation/attribution
   - *Impact indeterminacy:* Open-ended due to non-deterministic actions, agency of utilized models (tool-use chains), and opaque LLM supply-chains (base model → fine-tune → RLHF → system prompt — each layer opaque)
   - *User indeterminacy:* The operating skill floor has dropped sharply — offensive capability is no longer limited to trained practitioners

2. **Analyze moral attribution:** Who's responsible when an autonomous agent causes harm during a penetration test? The user who launched it? The tool-maker? The model provider? The paper argues responsibility becomes "diffuse" — spread across multiple parties with no clear locus.

3. **Stakeholder impact analysis:** Maps how agentic offensive security affects defenders, attackers, regulators, tool vendors, model providers, and the general public.

4. **Stratified recommendations:** Separate guidance for different stakeholder groups.

## Key Results (with numbers)

- Not an empirical paper — no quantitative results. The key analytic result is the **three-axis indeterminacy framework** for classifying agentic security tools.
- Key structural insight: **cost asymmetry between offense and defense** is amplified by agents, creating a net short-term advantage for attackers
- Long-term upside: same technology may democratize defensive practice (lowering the skill floor for blue teams too)

## What's Novel

- First paper to explicitly frame LLM-driven offensive agents as a distinct category with **emergent properties** not present in either traditional pen-testing tools or general AI ethics
- The "three indeterminacies" framework is original and practically useful for risk assessment
- Identifies the **attribution problem** for autonomous security agents — if an agent chain involves a model, a tool, a user, and a vendor, who's at fault when it goes beyond authorized scope?
- Connects to broader AI supply-chain security concerns

## My Connection (to Manny's Work)

This paper directly frames the ethical and operational context for Manny's own work as an AI red-team agent. The "three indeterminacies" are constraints Manny operates under: action indeterminacy affects explainability of findings; impact indeterminacy affects scope containment; user indeterminacy affects who can deploy Manny-like agents. The attribution analysis is directly relevant if Manny's actions cause unintended consequences during red-team exercises.

## What I Learned (Plain English)

Offensive AI agents aren't just "faster pen-testing" — they change the ethics and accountability structure of security testing. Traditional frameworks assume a human operator is in control and can explain their actions. With agents, the operator might not know why the agent did something, might not be able to predict what the agent will do next, and might not even be a security professional. The paper argues we need new frameworks that explicitly acknowledge this "loss of determinacy" rather than trying to fit agents into old models. For practitioners: document your agent's supply chain (base model, fine-tune weights, system prompt, tool definitions) and set scope boundaries that account for the agent's autonomy.
