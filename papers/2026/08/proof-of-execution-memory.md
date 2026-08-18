# Proof-of-Execution Memory: Defending LLM Agents Against Forged-Reasoning Attacks by Verifying What Actually Happened

- **Link:** https://arxiv.org/abs/2608.16032
- **Authors:** Md Habibur Rahman, Jaeho Kim
- **Institution:** Not stated in listing
- **Published:** Aug 17, 2026
- **Code:** https://github.com/bithabib/ai_security (8 pages, 6 figures, 5 tables)

## Problem Statement (plain English)

LLM agents are stateless — they carry context between steps in external "reasoning memory," and they trust that memory. The FARMA attack exploits that trust: an adversary who can write to memory inserts fabricated entries claiming a required safety step is already done, so the agent skips it. No malicious command is ever issued. The proposed defense at the time (SENTINEL) scores entries against a fixed list of suspicious wordings — trivially bypassable if the attacker knows the list.

## Methodology (technical)

- **Attack extension:** an automated attacker asks an LLM to *reword* the forged memory entry, testing whether rewording defeats SENTINEL's keyword list.
- **Capability paradox study:** attack success measured across model strengths (GPT-4o, GPT-4o-mini vs Llama-3.1-8B).
- **PoEM defense:** stop inspecting memory content entirely. Keep a separate, tamper-evident **HMAC-chained ledger** of safety steps that *actually executed*, writable only by the trusted action layer. A skip is allowed only if the ledger confirms real execution. Attackers can change what memory *says* but cannot forge a ledger entry for a step that never ran.
- Evaluated on 3 models × 3 scenarios, plus a real LangChain agent integration.

## Key Results (with numbers)

- Reworded forgery **evades SENTINEL on the first try** — protection drops to zero on every model tested.
- Capability paradox: **98–100% attack success on GPT-4o / GPT-4o-mini** vs **44% on Llama-3.1-8B** — stronger agents follow reworded claims more faithfully, so the threat grows with capability.
- **PoEM: attack success → 0%** on all models/scenarios; **0% false positives in 8 of 9 cells** (1.7% in the ninth, within noise). SENTINEL, by contrast, wrongly blocks 33–50% of legitimate operations.
- PoEM withstands attacks aimed at the ledger itself, adds **microseconds** of overhead, and works unchanged in a real LangChain agent.

## What's Novel

Moves the security boundary from *content inspection* (defeatable by paraphrasing) to *execution provenance* (an append-only, HMAC-chained record of what actually ran). This is a structural fix — it doesn't matter how the attacker words the lie, because the ledger only records facts. Also documents the counterintuitive result that capability scaling makes forged-reasoning attacks *more* effective, not less.

## My Connection (to Manny's work)

Memory-injection / forged-reasoning attacks are exactly the kind of thing Manny's agent engagements should test for: any agent with a memory store (MCP memory servers, reasoning logs, tool-result caches) is a candidate. PoEM is also a concrete defensive pattern worth recommending — verify execution provenance rather than trusting recorded reasoning — and a nice template for Manny's own agent tooling.

## What I Learned (plain English)

- An attacker doesn't need to issue commands to an agent; they can just edit what the agent *believes* happened.
- Keyword-list defenses against paraphrased attacks are worthless — an LLM can reword a lie on the first try.
- The smarter the model, the more reliably it obeys a forged memory entry — capability is not safety.
- Defend by recording ground truth (what executed) instead of judging claims (what memory says).
