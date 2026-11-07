# The Promise of Strategic Agentic Scambaiting

**Authors:** Abhishek Satyam, Hossein Siadati, Haadi Jafarian, Sima Jafarikhah, Juntao Chen
**Published:** Submitted November 5, 2026 (arXiv:2611.08170)
**Link:** https://arxiv.org/abs/2611.08170
**Length:** 18 pages

---

## Problem Statement

Online fraud has become an **agent-to-agent** contest and almost nobody has modeled it as one. On the offensive side, scammers have moved from automated message generation to fully agentic conversational systems that scale operations and extract maximum value from fraudulent resources before detection and takedown. On the defensive side, "scambaiting" has professionalized: autonomous agents impersonate viable victims, hold scammers in sustained conversation, and extract actionable intelligence — mule bank accounts, crypto wallet addresses — that can be reported to disrupt the infrastructure.

The unexplored question: **in an environment where both sides are agents, under what conditions does defensive baiting actually work, and what decides who wins?**

## Methodology

The authors build a formal model of agent-to-agent fraud as a strategic interaction, capturing the decision spaces available to attacker and defender.

The model tracks four coupled dynamics:

1. **Engagement** — whether and how long each side continues an interaction.
2. **Resource consumption** — the cost to the scammer of maintaining a conversation, and the cost to the defender of sustaining a persona.
3. **Belief adaptation** — each side updating its estimate of whether the other is what it claims.
4. **Disruption outcomes** — whether extracted intelligence converts into infrastructure takedown.

The key strategic variables are **baiter prevalence** (how much of the inbound traffic is bait) and **mimicry quality** (how convincingly a defensive agent sustains a victim persona) against **scammer detection capability**.

## Key Results

- **Scammer profitability turns negative once baiting-agent prevalence exceeds a threshold.** Baiting is not just intelligence collection; at sufficient density it is an economic denial-of-service on fraud operations.
- **When both sides improve at the same pace, defense wins.** Defender investment in *mimicry quality* outpaces equivalent gains in scammer *detection* capability, so the arms race favors the defender under the paper's baseline parameters.
- The advantage is parameter-dependent, not unconditional — the framework is explicitly a way to locate the thresholds rather than a claim that baiting always wins.

## What's Novel

1. **Treats scambaiting as a strategic game rather than a case study.** Prior work is anecdotal ("here is a honeypot that wasted a scammer's time"). This gives engagement length, resource burn, and takedown conversion a formal structure.
2. **Identifies prevalence as a control variable.** The density of bait in the inbound pool changes attacker economics — which implies a collective-action argument for shared baiting infrastructure.
3. **Asymmetry result.** Mimicry quality is cheaper to improve than detection, which is a non-obvious claim about where defensive investment should go.

## My Connection

- **Threat-intel collection architecture.** If Manny runs any adversary-engagement or deception capability, this model says the value is in *density and persistence*, not individual conversations — one great bait session is worth less than a steady presence.
- **Attacker-side reading.** Flip it: the same model tells you how a competent fraud operation detects bait (resource-consumption anomalies, persona inconsistency under belief adaptation) and how it should triage inbound leads. That is directly useful when red-teaming a fraud target or advising on detection.
- **Agent-vs-agent eval design.** The four dynamics — engagement, resource consumption, belief adaptation, disruption — are a reasonable scaffold for evaluating any persistent adversary agent, including the always-on "Teammates"-style agents shipping this week.

## What I Learned

Scambaiting sounds like a hobby, but modeled properly it is an economic attack on fraud infrastructure: the lever is not what you extract from one scammer, it is what you cost them per unit of inbound traffic. The broader lesson is that the interesting security questions about agents are increasingly *economic* — who pays per interaction, who can afford to keep the conversation going, and which side's marginal improvement is cheaper. That framing transfers well beyond fraud, to spam, to scraping, and to agent-vs-agent denial of service.
