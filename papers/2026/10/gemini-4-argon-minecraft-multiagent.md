# A team of Gemini 4 Argon agents beats Minecraft

**Authors:** Maxence Faldor, Satinder Singh, Roberta Raileanu
**Institution:** Google DeepMind (author affiliation; arXiv cs.AI)
**Date:** Submitted Oct 15, 2026 (announced Oct 19, 2026; broad press coverage Oct 19)
**Link:** https://arxiv.org/abs/2610.22319
**Code:** Not stated in the abstract (uses the open-source Mineflayer library)

---

## Problem Statement (plain English)

Long-horizon agent work is easy to demo and hard to prove. Benchmarks are usually short, scaffolded, and human-designed. Minecraft survival is a useful stress test because it requires planning across thousands of dependent steps, reacting to a changing world, and coordinating multiple agents — with no human writing game logic. The question: can a team of frontier agents go from a random world to the final boss with only a general-purpose library and no help?

## Methodology (technical)

- **Setup:** four Gemini 4 Argon agents in a sandboxed, randomly generated Minecraft survival world on Easy difficulty.
- **Interface:** each agent controls a player through **Mineflayer**, an open-source JavaScript Minecraft client library. No game-specific human-written tools and no pre-scouted seeds.
- **Coordination:** agents talk through in-game chat and a shared memory, with no assigned roles. They plan, write, and debug their own programs.
- **Oversight:** researchers watched a live dashboard and could pause or stop the run at any time, but did not intervene.
- **Analysis:** the paper documents the run timeline, the team's strategies (work division, sharing learned procedures, recovering each other's dropped items), and the limits of the result.

## Key Results (with numbers)

- **27 hours 14 minutes** from spawn to Ender Dragon kill.
- **2,783** programs written and executed by the agents.
- **605** deaths during the run.
- **4** agents, **0** assigned roles, **0** lines of human-written game logic.
- Claimed as the **first end-to-end autonomous Minecraft completion** using only standard Mineflayer, without pre-scouted seeds or bespoke tooling.

## What's Novel

- End-to-end autonomy over a **multi-day** horizon rather than a scripted benchmark episode.
- Emergent division of labor and peer teaching through shared memory, with no role assignment.
- A public, inspectable environment that stresses the same primitives as enterprise agent work: long plans, tool use, recovery from failure, and multi-agent coordination.
- Honest reporting of failure volume (605 deaths) alongside the success — the cost of long-horizon autonomy is visible.

## My Connection (to Manny's work)

This is the capability baseline that makes the security findings on the same day urgent: agents that can hold a coherent plan for 27 hours and write 2,783 programs are exactly the agents that will be given repo write access, shell tools, and approval workflows. It also gives a clean observability target — shared memory, chat coordination, and program logs are the artifacts you would need to audit if this were an enterprise agent team instead of Minecraft.

## What I Learned (plain English)

Four agents with no assigned roles can teach each other, split work, and finish a day-long task — but they died 605 times doing it. Long-horizon autonomy is real, and the measurable cost is failure volume plus the tokens and time to recover. The security question is not whether agents can do this; it is what they can reach while they are doing it.
