# Emerging Challenges in Threat Modeling for GenAI-Augmented Systems: A View from the Trenches

- **Paper:** https://arxiv.org/abs/2607.28431
- **Authors:** Nicolás E. Díaz Ferreyra, Manish Mahesh Kumar, Nohemí Villarreal, Pankaj Pantel, Immo Brueggemann, Riccardo Scandariato
- **Institution:** Not stated on abstract page (accepted at ESEM 2026)
- **Published:** Jul 30, 2026 (arXiv cs.SE / cs.CR)
- **Code:** Not stated in abstract

## Problem Statement (plain English)

Threat modeling (e.g., STRIDE) is how security teams find problems from architecture diagrams before code ships. But as GenAI gets bolted onto ordinary software systems, the classic methods miss the new risk categories — prompt injection, model poisoning, data leakage through context, and so on. Everyone assumes you can just "add AI to STRIDE," but nobody had actually tested which GenAI-aware methods work in a real company, with real constraints (small team, limited time, shipping pressure).

## Methodology (technical)

The authors ran an exploratory assessment in a Small and Medium Enterprise (SME) setting:

1. **Rapid literature review** to shortlist GenAI-aware threat modeling techniques.
2. **Systematic application** of three shortlisted methods to an industrial case study — a real GenAI-augmented system in the SME.
3. **Practitioner evaluation** of usability, integration into existing dev workflows, perceived effort, and adoption barriers.

## Key Results

- The three techniques surface **different and partially non-overlapping threat sets** — no single method was sufficient on its own.
- **Limited coverage of GenAI-specific risk categories**, particularly **software supply chain** risks and **human-centered security** issues (social engineering, human-in-the-loop failure modes).
- Practitioners reported real friction: perceived effort and integration challenges in SME workflows.

## What's Novel

- First empirical, industry-grounded evaluation of GenAI-aware threat modeling — not another proposed framework, but a test of existing ones in the trenches.
- Concrete evidence that current methods under-serve supply-chain and human-centered GenAI risks — two categories that dominate real-world AI incidents.
- Actionable gap list for tooling builders and method designers.

## My Connection (to Manny's work)

Manny's red-team engagements end with threat models and recommendations — this paper tells him which existing GenAI threat-modeling methods actually hold up in practice and where they go blind (supply chain, human factors). It's a ready-made lens for structuring findings for SME clients, and the gap analysis doubles as a checklist: if a client's threat model doesn't cover AI supply chain and human-centered risks, flag it.

## What I Learned (plain English)

There's no "one weird trick" for GenAI threat modeling yet — real teams need to combine methods, and even then, the two risk areas that hurt most in practice (supply chain and human factors) are the ones current techniques cover worst. Also: a method only counts if a busy team will actually use it; academic completeness loses to integration and low effort.
