# AKRASIA: Stealthy Backdoor Attack on Reasoning-based Code LLMs

- **Paper:** AKRASIA: Stealthy Backdoor Attack on Reasoning-based Code LLMs
- **Link:** https://arxiv.org/abs/2609.01023 (HTML: https://arxiv.org/html/2609.01023v1)
- **Authors:** Chou Jin Chua, Sarang Nambiar, Murali Srinivasan, Ezekiel Soremekun (per arXiv cs.CR listing)
- **Institution:** Not confirmed at digest time (author-affiliation data not surfaced)
- **Published:** September 1, 2026 (arXiv:2609.01023, cs.CR)
- **Code:** Not confirmed; a GitHub repo matching the akrasia theme surfaced in search (Chandrachud23/LLM-Akrasia) but is not verified as this paper's code

## Problem Statement (plain English)

Reasoning code LLMs don't just emit code — they emit a chain of thought ("first I'll parse the input, then…") and then the code. That makes them harder to attack with classic backdoors: a poisoned model that suddenly writes malicious code usually gets caught because its *reasoning looks wrong* or a defense inspects the final code. AKRASIA asks: can you plant a backdoor that only fires on a specific code-level trigger, produces malicious behavior, yet keeps the model's stated reasoning perfectly plausible — so neither automated defenses nor human reviewers notice? And can you do it *without fine-tuning*, so weight-level scans find nothing?

## Methodology (technical)

- **Inference-time backdoor, no training-time poisoning.** AKRASIA first *probes the victim LLM* to construct a code-level backdoor trigger (a code pattern the target model reliably responds to).
- **In-context learning for backdoor learning:** the trigger→malicious-behavior association is taught through crafted demonstrations in the context, not through gradient updates — evading weight-inspection defenses.
- **Exploits model unfaithfulness:** AKRASIA leverages the gap between a model's stated reasoning and its actual output ("akrasia" = weakness of will) to conceal the backdoor trigger and *generate plausible reasoning* — the chain-of-thought narrates a benign approach while the emitted code executes the backdoor target (e.g., malicious code execution).
- **Evaluation:** four backdoor targets, six reasoning LLMs, three coding task datasets, and three defense methods.

## Key Results (with numbers)

- Works across **six reasoning code LLMs** and **three coding task datasets** with **four different backdoor targets**.
- Evades **three defense methods** tested, while keeping reasoning plausible enough to pass human inspection (the paper's central claim).
- Demonstrates a concrete code-supply-chain scenario: a "helpful" reasoning model that, on trigger, generates code with a hidden malicious behavior without its reasoning ever flagging it.

## What's Novel

Backdoors usually mean poisoned training data or fine-tuning — detectable via data audits or weight analysis. AKRASIA is a *pure inference-time* backdoor that uses the model's own reasoning against it: probing to build the trigger, ICL to install the behavior, and unfaithful chain-of-thought to launder the attack past reviewers. It targets the one thing coding agents now trust most — the model's visible reasoning — as the camouflage.

## My Connection (to Manny's work)

Coding agents (Claude Code, Codex, and the harnesses in today's Context-PE paper) are the highest-trust tools Manny red-teams. AKRASIA is the supply-chain nightmare behind them: a model or scaffolded agent that *sounds* safe while emitting trigger-activated malicious code — no poisoned weights to find. It sharpens the case for output-level and behavior-level monitoring (what the code does, not what the model says) and for the kind of agent-skill/MCP vetting AIR is commercializing. Also relevant to any eval Manny runs that treats chain-of-thought as ground truth.

## What I Learned (plain English)

A reasoning model's visible thought process is not evidence of its intent — and attackers can deliberately exploit that gap. You can plant a backdoor without ever touching the model's weights: probe to find a trigger, teach the trigger-behavior link through examples in the context, and let the model's own unfaithful reasoning supply the alibi. Defending means watching the *executed output* and runtime behavior, not trusting the narration — and treating every model-integration point (including in-context scaffolding) as a potential backdoor installation site.
