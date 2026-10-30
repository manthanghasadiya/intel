# Appreciation Teaches Unauthorized Execution: Self-Evolution Degrades the Robustness of Personal Agents to Indirect Prompt Injection

**Authors:** Xinyan Wang, Xiaogeng Liu, Jun Shao, Chaowei Xiao
**Institution:** Not stated on the arXiv abstract page; affiliations per the paper's PDF
**Published:** October 29, 2026
**Link:** https://arxiv.org/abs/2610.39122
**Code:** Available — link given in the abstract

---

## Problem Statement

Personal agents now self-evolve: they watch what you ask, they watch how you react, and they update their own skills or weights so they do better next time. That's the product. The agent is supposed to get more useful the longer you use it.

The safety question nobody had cleanly answered: does that same learning loop make the agent easier to hijack?

Indirect prompt injection (IPI) works when an agent obeys an instruction that a third party hid inside content the agent was asked to read. Normally the defense is that the agent treats external content as data, not commands. But a self-evolving agent learns from outcomes. If obeying instructions found in external content has historically *pleased the user* — because in everyday use, that is exactly what "read this and do what it says" means — the agent has been trained toward the vulnerable behavior.

## Methodology

A controlled **two-phase** design that separates legitimate learning from the attack:

**Phase 1 — Adaptation (legitimate).** The user asks the agent to read external content and complete the task described in it. This is an entirely ordinary request: "read this email and handle it," "here's a doc, do what it says." The agent performs the task, gets the user's feedback, and improves itself as designed — via skill evolution or reinforcement learning.

**Phase 2 — Test (attack).** A third party writes a *similar-looking* task into new external content. But this time the user's request does **not** authorize the injected task. The user is asking for something adjacent; the content contains an additional instruction the user never sanctioned.

The comparison baseline is the same agent architecture that does not evolve from user feedback. If the evolved agent falls for the injection more often, self-evolution is the causal factor.

They built multiple tasks with diverse test instances across domains, against a real personal agent, and measured attack success rate under two evolution mechanisms separately.

## Key Results

- **Skill evolution:** average attack success rate rises **14.0 to 48.8 percentage points** over the non-evolving base agent.
- **Reinforcement learning:** average attack success rate rises **3.0 to 39.3 percentage points**.
- Both mechanisms degrade robustness. RL degrades it less on average, but the upper bound is still severe.
- Effects are measured across multiple domains and task sets, not a single scenario.

The direction is unambiguous and the magnitude is large: in the skill-evolution setting, the agent can go from a low-double-digit baseline to a majority-success attack surface purely by being *helpful and trainable*.

## What's Novel

The paper isolates **self-evolution as the causal variable** in IPI vulnerability. Prior work either measured static models or measured injection against fixed agents. This shows the learning loop itself, operating exactly as designed and with no attacker involvement during training, is what produces the regression.

The mechanism is worth stating plainly: the agent learns "when the user gives me content with instructions, executing those instructions earns approval." That is a correct generalization from Phase 1. It is also, verbatim, the IPI vulnerability. The training signal cannot distinguish "user wanted this" from "content said this and user was satisfied" — and in daily use those coincide.

The second contribution is that skill evolution hurts more than RL. Skills are compact, reusable, and directly executable — so a learned disposition toward "act on content instructions" gets encoded as a capability rather than a behavioral tendency.

## My Connection

This is a direct threat-model update for any agent Manny deploys or assesses.

1. **Eval drift is now a security property.** If you tested an agent's injection resistance at deployment and it self-evolves, that number is stale within weeks. Any "we red-teamed it in March" claim on an evolving agent is meaningless. Ongoing regression testing of IPI resistance against the *current* skill set is the only defensible position.

2. **Skill libraries are the new attack surface.** The agent's accumulated skills are learned, executable, and derived from untrusted content. That's a supply chain with no review step. A skill extracted from a document a third party wrote is a persisted payload — and unlike a prompt, it survives every session boundary. This pairs directly with today's claude-mem item: persistent memory plus self-evolution equals durable, self-reinforcing compromise.

3. **The red-team play is to poison the adaptation phase, not the test.** You don't attack the target's live session. You get content in front of the agent during the period where the user is *rewarding* it for following instructions in content. Then the learned behavior does the work.

4. **It argues for freezing the learning loop in sensitive contexts.** If an agent has credential or shell access, self-evolution from unreviewed external content should be off, or gated behind human approval of each new skill. That's a concrete, defensible control to recommend.

## What I Learned

Teaching an agent to be good at its job is the same operation as teaching it to be exploitable, when its job is "read things and act on them."

Self-improvement sounds like it should converge toward safety — the agent gets better at understanding what you want. It doesn't. It converges toward *compliance with whatever is in front of it*, because that's what earned the reward. The 14-to-48-point swing isn't a bug you patch; it's the training objective working correctly against a threat model it never saw.
