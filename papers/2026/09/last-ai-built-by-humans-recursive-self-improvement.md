# The Last AI Built by Humans: Toward Genuine Recursive Self-Improvement

**Authors:** Yi Duan, Ying Liu, Zirui Tang, Haodong Chen, Jun Zhou, Yumou Liu, Bangrui Xu, Yukai Wu, Sidi Chen, Yuhan Zhou, Haoyu Wang, Xiaoyou Yu, Shaokun Han, Xuzhou Zhu, Le Zhou, Bolin Lu, Wei Zhou, Jiachen Liu, Nuozhou Fang, Jiaxin Tian, Ruoyu Chen, Yuxuan Li, Kai Zuo, Kaiyan Zhang, Jiantao Qiu, Conghui He, Guoliang Li, Bowen Zhou, Zhiyuan Liu, Zhoufutu Wen, Jihua Kang, Xuanhe Zhou, Fan Wu (33 authors)
**Institution:** Large multi-institution effort; corresponding group includes Tsinghua University–affiliated researchers (Zhiyuan Liu, Jiantao Qiu, Conghui He) — treat affiliation as unconfirmed pending the full PDF.
**Date:** September 10, 2026
**Link:** https://arxiv.org/abs/2609.11873
**Code:** Not stated in abstract

---

## Problem Statement (plain English)

"Recursive self-improvement" (RSI) is the idea that an AI can use its own experience and feedback to make itself permanently better — and improve *the process* by which it improves. Everyone talks about it; almost nobody has a rigorous way to say where today's models actually stand or what the missing pieces are. This paper asks a sharper version of the question: how do we *measure* the headroom, and what's the concrete roadmap from today's LLMs to genuine RSI?

## Methodology (technical)

The paper is a **conceptual framework plus empirical diagnosis**, not a single-model experiment:

1. **Headroom-Closed Index (HCI)** — a metric introduced to reveal where existing LLMs plateau on self-improvement, i.e., how much of the theoretical improvement headroom is actually being closed.
2. **A five-stage RSI roadmap**, defined as increasing autonomy:
   - improvement-**execution** autonomy (can it *carry out* a given improvement?)
   - improvement-**strategy** autonomy (can it *choose* what to improve?)
   - experience-**acquisition** autonomy (can it *generate the data/experience* it needs?)
   - environment-**adaptation** autonomy (can it *change its environment* to improve?)
   - recursive **meta-improvement** (can it improve its own improvement process?)
3. **Cross-domain analysis** — RSI requirements and development speeds compared across scientific discovery, embodied intelligence, and software engineering, which have very different bottlenecks.
4. **Industry-practice grounding** — connects the framework to real deployed systems and preliminary empirical evidence.

## Key Results

- The **HCI reveals systematic problems with existing LLMs** when it comes to self-improvement — i.e., current models sit well short of closing the headroom, and the index makes that quantitative rather than hand-wavy.
- The five autonomy stages are shown to be **distinct and unevenly progressed** — execution autonomy is comparatively near, while strategy/acquisition/adaptation and meta-improvement are progressively further out.
- Different domains advance at **different speeds**: software engineering and scientific discovery move faster than embodied intelligence, because their feedback loops are cheaper and more legible.
- The paper's central deliverable is a **map of the key challenges** between today's systems and genuine RSI, connecting research to practical systems rather than declaring a capability.

## What's Novel

- **The HCI metric** gives the RSI debate an actual measurement instrument instead of a vibes argument.
- The **staged autonomy decomposition** is a cleaner taxonomy than the usual binary "can it self-improve / can't it" framing — it tells you *which* autonomy is missing.
- Explicitly **linking RSI research to deployed systems and industry practice** (33 authors spanning multiple labs/institutions) rather than keeping it in the abstract.
- Cross-domain comparison of *development speed*, not just feasibility.

## My Connection

RSI is the capability story everyone is building toward (see OpenAI's "automated research intern" milestone and the Navier–Stokes agent swarm in this week's news feed). For red-teaming, the HCI-style decomposition is useful the other way round: each autonomy stage is also a **new attack surface**. Improvement-execution autonomy means an agent that can write and deploy code to itself; environment-adaptation autonomy means an agent that can modify its own harness. Those are precisely the boundaries the Anthropic incidents and the RubyGems/ADK stories show failing. Manny can use this roadmap as a checklist of "which autonomy am I granting, and what's the containment for it."

## What I Learned (plain English)

Self-improving AI isn't one switch — it's a ladder. Today's models can mostly *do* an improvement you hand them; they can't yet decide *what* to improve, go *get* their own experience, or rewrite the rules they improve by. The paper's real contribution is a ruler (HCI) and a ladder, so we can stop arguing about whether AGI-style self-improvement "has happened" and instead measure how many rungs we've actually climbed.
