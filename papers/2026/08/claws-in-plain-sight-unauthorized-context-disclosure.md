# The Claws in Plain Sight: Unauthorized Context Disclosure through LLM Agent Tool Calls

- **Link:** https://arxiv.org/abs/2608.20658
- **Authors:** Ben Dong, Zhonghao Guo, Tianyi Lu, Qian Wang
- **Institution:** Wuhan University (Qian Wang, School of Cyber Science and Engineering)
- **Published:** Aug 24, 2026 (submitted Aug 21, 2026)
- **Code:** Not released (per abstract page)

## Problem Statement (plain English)

LLM agents build tool-call arguments from whatever context they have: user profiles, chat history, retrieved documents, prior tool outputs. The catch: *having access* to a piece of context (like a user's SSN, email, or location) doesn't mean the agent is *authorized to transmit it* in every tool call. Attackers can smuggle "task-adjacent" instructions into the context that frame protected attributes as operationally or procedurally required — and the model dutifully includes them in otherwise-valid tool arguments, exfiltrating data through legitimate-looking API calls.

## Methodology (technical)

- Attack: "authority-pressure" — craft task-adjacent content that makes protected attributes look required for the task, causing the model to embed them in generated tool-call arguments.
- Controlled synthetic benchmark crossing **6 pressure levels** × **4 privacy-policy levels** across **5 model configurations** (DeepSeek + Claude family), producing **120 tool calls**.
- Measures session-level disclosure rates across the full pressure-policy matrix, plus the effect of stronger privacy instructions.

## Key Results (with numbers)

- Session-level disclosure rates range **20.8% to 75.0%** across tested models/configurations.
- Stronger privacy instructions reduce leakage but do **not** eliminate it — the attack remains viable even under strict policy framing.

## What's Novel

- Targets *tool-call argument construction* as the disclosure channel, rather than the usual prompt-injection/exfiltration-in-response framing.
- The "authority-pressure" mechanism: legitimate task framing is the weapon, so the model's outputs look valid and pass downstream checks.

## My Connection (to Manny's work)

This is a ready-made red-team technique: when testing enterprise agents, inject authority-pressure context and inspect tool-call arguments (not just final responses) for protected attributes. The 20.8–75% leakage band is the headline stat to quote when clients claim their guardrails handle data exfiltration.

## What I Learned (plain English)

Agent data loss has a quieter channel than the dramatic jailbreak: the model just includes the sensitive field because the context made it look necessary. Monitoring tool-call *arguments* — not just outputs — is a non-negotiable detection layer, and "privacy policy" prompts are a weak control on their own.
