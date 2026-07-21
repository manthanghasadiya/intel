# Insecure Coding Preferences in Long-Term Memory: Security Risks for LLM-based Code Generation

- **Link:** [https://arxiv.org/abs/2607.17619](https://arxiv.org/abs/2607.17619)
- **Authors:** Yuchen Chen, Wei Cheng, Yuan Xiao, Zhou Yang, Weifeng Sun, Chunrong Fang, Xiang Chen, Baowen Xu, David Lo, Zhenyu Chen
- **Institution:** Southeast University, Nanjing University, Singapore Management University, Waseda University
- **Published:** July 20, 2026
- **Code/Dataset:** Not explicitly mentioned in abstract

## Problem Statement (Plain English)

LLM-based coding assistants (ChatGPT, Gemini, Qwen, Grok) now support long-term memory to maintain context across sessions. If a user pastes insecure code (or an attacker poisons the memory), the model may silently learn those insecure patterns and generate vulnerable code in future sessions. The question: how bad is this risk, and can we fix it?

## Methodology (Technical)

1. **Setup:** Evaluated 4 LLMs (ChatGPT, Gemini, Qwen, Grok) × 5 programming languages (Python, C, C++, Go, JavaScript).
2. **Memory Poisoning:** Injected insecure coding patterns into each model's long-term memory (e.g., SQL injection patterns, unsafe memory management, hardcoded credentials).
3. **Measurement:** Tracked vulnerability rate, warning rate (model's own security warnings), and the "risk-warning gap" (how much warnings lag behind actual vulnerabilities).
4. **Mitigation Evaluation:** Tested three strategies — security-requirement appending, memory storage with security filtering, and memory-level safety filtering.

## Key Results

- Insecure memories increase vulnerable code generation by **2.7–50.3 percentage points** (varies by model and language).
- Creates a **5.4–14.0 pp risk-warning gap** — the model's own warnings increase slower than actual vulnerabilities, meaning users get a false sense of security.
- Insecure memories are **hard to overwrite** through normal interactions; simple correction prompts don't reliably fix the stored pattern.
- Memory influence is **broad** — affects outputs even when prompts are phrased differently.
- **Memory-level safety filtering** achieves 100% detection on evaluated risky entries and restores generation to the without-memory baseline.
- Security-requirement appending reduces vulnerability rates by 19.7–33.6 pp but degrades functional correctness by up to 15.9 pp.

## What's Novel

- First systematic study of **long-term memory poisoning** in LLM code generation (not prompt injection, not adversarial inputs — persistent memory corruption).
- Quantifies the "risk-warning gap" as a measurable security metric.
- Proposes and validates memory-level safety filtering as the most effective mitigation (100% detection, no functional correctness loss).

## My Connection (to Manny's Work)

- Directly relevant to mcpsec: if an agent framework uses long-term memory (most do — Cursor, Codex, Claude Code), it's vulnerable to persistent poisoning.
- The 100% detection rate of memory-level safety filtering suggests a concrete mitigation to implement in mcpsec's agent security layer.
- The finding that insecure memories are hard to overwrite through normal interaction means that a single poisoning event has long-lived effects — this is a supply-chain risk for agent tool configurations stored in memory.
- The risk-warning gap (vulnerabilities increase faster than warnings) means naive monitoring is insufficient.

## What I Learned (Plain English)

Long-term memory in LLMs isn't just a convenience feature — it's a new persistence surface for security attacks. Once an insecure coding pattern gets stored (deliberately by an attacker or accidentally by a developer), it silently influences all future code generation in that session. The scary part: even if the model warns about some vulnerabilities, the warning rate doesn't keep pace with the actual vulnerability rate, so users think they're safer than they are. The good news: memory-level filtering catches 100% of risky entries with no correctness loss. The takeaway: any agent framework with persistent memory needs a memory sanitization layer.
