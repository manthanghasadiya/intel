# StepJack: Benchmarking Computer-Use Agent Safety Against Multi-Step Indirect Prompt Injection

- **Authors:** Zhuoxin Zhan, Akbar Rafiey, Avery Ma, Leila Pishdad, Layla El Asri
- **Institution:** Not listed on arXiv abstract page (author group associated with Layla El Asri's industry research team)
- **Published:** Submitted Aug 6, 2026; announced Aug 10, 2026
- **Link:** https://arxiv.org/abs/2608.06477
- **Code:** Dataset and code released (link available on the arXiv abstract page)

## Problem Statement (plain English)

Computer-use agents (CUAs) browse the web, read pages, and act on what they see. Indirect prompt injection is the attack where a *web page itself* (or an email, doc, comment) contains hidden instructions that hijack the agent. The classic defense assumption is that a single injected instruction looks suspicious — it's a big, obvious ask. This paper shows attackers don't need to make one big ask: they can chop a dangerous goal into many tiny, individually harmless-looking steps and scatter them across a chain of pages the agent visits naturally. Each step looks innocent; the chain does the damage.

## Methodology (technical)

- **Decomposition pipeline:** Given an adversarial goal (e.g., "exfiltrate a file"), the pipeline automatically decomposes it into sub-steps under the constraint that executing all sub-steps achieves the original goal, while optimizing each sub-step for "innocuousness" (how harmless it looks in isolation).
- **Benchmark construction:** 480 test examples forming the StepJack benchmark, with attacks distributed across chains of pages along the agent's navigation path.
- **Evaluation:** Six state-of-the-art computer-use agents tested at varying decomposition depths (single-step vs multi-step).

## Key Results (with numbers)

- Multi-step attacks raised attack success rate (ASR) on 3 of 6 CUAs at a fixed decomposition depth, by **up to 31.2 points**.
- **GPT-5.4-mini: 41.7% ASR at single-step → 72.9% at three-step.**
- Averaged over the five CUAs that reliably follow the reference chain (all but EvoCUA-32B): ASR rose from **31.3% (single-step) to 36.9% (three-step)**.
- EvoCUA-32B could not reliably follow reference chains, so multi-step attacks don't transfer everywhere — the attack's effectiveness depends on the agent's instruction-following reliability.

## What's Novel

- First benchmark for **multi-step** indirect prompt injection, as opposed to single-shot injection.
- An automated goal-decomposition pipeline with explicit innocuousness optimization — attackers get a tool to generate distributed attack chains.
- Quantifies that decomposition depth is a lever: safety evals that only test single-step injection systematically underestimate real-world risk.

## My Connection (to Manny's work)

Directly actionable for agent red-teaming: StepJack's decomposition approach is exactly the technique to try against Claude Code's new auto-mode classifier (default Aug 14), MCP-based agents, and any per-call safety filter — if a classifier scores individual tool calls, decomposed chains that pass each call individually are the natural bypass. Also a ready-made eval harness pattern for Manny's own agent-safety testing.

## What I Learned (plain English)

Safety checks that look at one action at a time can be defeated by spreading a bad plan across many good-looking actions. Any agent red-team or eval suite should test attacks at multiple "depths," not just single prompts — and the more reliably an agent follows instructions, the more reliably it follows an attacker's chain too.
