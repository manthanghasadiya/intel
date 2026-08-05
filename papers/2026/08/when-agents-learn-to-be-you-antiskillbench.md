# When Agents Learn to Be You: Benchmarking Privacy Leakage, Impersonation Risk, and Defenses in Persona Skills

- **Paper:** https://arxiv.org/abs/2608.03700
- **Authors:** Yongli Xiang, Zhifang Zhang, Bojun Yang, Ziming Hong, Lei Feng, Miao Xu, Tongliang Liu
- **Institution:** (Sydney-based group; Tongliang Liu is University of Sydney)
- **Published:** Aug 4, 2026
- **Code:** N/A (benchmark: AntiSkillBench)

## Problem Statement (plain English)

"Persona skills" — distilled, portable artifacts that let an agent act like a specific person (their tone, habits, preferences) — are becoming a personalization primitive. The problem: to build one, you compress a person's interaction history into an executable file, which concentrates all their fragmented personal signals into one artifact that gets reused across agents. That's a privacy and impersonation bomb: the skill can leak attributes, and any attacker who obtains it can make an agent *be* that person. Existing defenses target individual records or retrieval-based memory, which don't map to this new artifact type.

## Methodology (technical)

AntiSkillBench, an end-to-end benchmark:

1. **Dataset:** 7,500 persona-grounded dialogue traces constructed from 50 behaviorally rich profiles spanning diverse task scenarios.
2. **Evaluation suite:** measures skill-level privacy leakage, agent-level attribute disclosure, and behavioral impersonation across three skill-distillation strategies.
3. **Defense evaluation:** four configurations across online and post-hoc interventions — active risk suppression (during distillation) and passive provenance protection (after).
4. **Backbones:** evaluated across three frontier agents to test generalization.

## Key Results (with numbers)

- Persona-skill risks **persist across agent backbones and distillation protocols**, extending from explicit attributes to communication style and personality traits.
- Existing defenses show **limited, distillation-dependent effectiveness** — they fail to generalize across risk types and distillation strategies.
- The takeaway: this is a hard, unsolved benchmark — no current defense reliably prevents leakage/impersonation through persona skills.

## What's Novel

First systematic treatment of the *persona-skill pipeline* as an attack surface, distinguishing it from plain memory retrieval (which is what prior privacy work studied). Introduces a reusable benchmark (AntiSkillBench) spanning distillation strategies, risk types, and defenses, and shows the failure of current defenses to generalize — establishing the problem as open rather than solved.

## My Connection (to Manny's work)

Any agent that personalizes — custom personas, memory distillation, user-profile skills — inherits this risk. For red-team work it's a concrete attack recipe: steal/prompt-craft a persona skill, then measure attribute disclosure and behavioral impersonation. For defensive work, it says provenance and active suppression during distillation matter more than post-hoc filtering. Directly relevant if Manny builds or audits agent personalization features.

## What I Learned (plain English)

Compressing a person's history into a skill file is like giving away a behavioral fingerprint — it leaks not just facts about them but *how they act*, and it generalizes across models. Defenses that work on raw memory records don't transfer to distilled skills because the distillation step reshapes the data. If you're going to build persona artifacts, the safety properties have to be designed into the distillation process itself, not bolted on afterward.
