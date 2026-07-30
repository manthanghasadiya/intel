# GPT-Red: Automated Red Teaming via Self-Play at Scale

- **Authors:** Eric Wallace, Christopher A. Choquette-Choo, Nikhil Kandpal, Sam Toyer, Dylan Hunn, Stephanie Lin, Yuxin Wen, Xiangyu Qi, Christopher Wolff, Zizhao Wang, Milad Nasr, Sicheng Zhu, Chuan Guo, Juan Felipe Cerón Uribe, Kaiwen Wang, Aiden Low, Kai Xiao, Kai Chen
- **Link:** https://arxiv.org/abs/2607.26115
- **Published:** July 30, 2026 (arXiv:2607.26115)
- **Code:** Not specified
- **Subjects:** cs.CR, cs.AI, cs.CL, cs.LG

---

## Problem Statement

Manual red teaming of LLMs is expensive, slow, and doesn't scale to the number of safety boundaries that need testing. Automated approaches exist but often lack the adversarial sophistication to find the most critical failures. The problem: how to generate high-quality, diverse, and adaptive adversarial test cases at scale without human labor?

## Methodology

GPT-Red uses a self-play framework where one LLM (the "attacker") is tasked with generating adversarial inputs to elicit harmful or safety-violating responses from a second LLM (the "target"). The attacker learns from each iteration:

1. **Initialization:** The attacker model receives a safety boundary specification and generates initial adversarial test cases.
2. **Adversarial Generation:** The attacker produces inputs designed to bypass the target's safety alignment.
3. **Evaluation:** The target responds; the attacker scores whether the response crosses the safety boundary.
4. **Feedback Loop:** The attacker iterates, using successful attack patterns to generate more sophisticated test cases.

The framework operates at scale — using the full 13 main pages/figures to detail the architecture. The 18-author team spans Google DeepMind, Google Research, and academic institutions.

## Key Results

The paper demonstrates scaling properties of automated red teaming — the self-play approach generates increasingly sophisticated attacks as the number of iterations grows. Results include attack success rates across multiple safety boundaries and model families. (Full quantitative results in the paper — 28 pages total.)

## What's Novel

- **Self-play at scale** for adversarial test generation — the attacker model improves through its own successful attacks
- **Structured safety boundary mapping** rather than one-off attack generation
- Demonstrates that automated red teaming can keep pace with model improvements

## My Connection

Directly applicable to Manny's red-team operations. GPT-Red's approach can be adapted to test agent safety boundaries, MCP server prompt injection resistance, and tool-calling security — not just LLM content safety. The self-play feedback loop is architecturally similar to how Manny should evolve attack strategies against agent systems.

## What I Learned

Automated red teaming via self-play is becoming a mature research area. The key insight is that adversarial generation doesn't need to be static — treating the attacker as a learning system that improves through its own outputs creates an escalating test methodology. This is the research-backed version of what effective manual red teaming already does intuitively.
