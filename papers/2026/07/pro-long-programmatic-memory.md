# Paper: PRO-LONG — Programmatic Memory Enables Long-Horizon Reasoning

- **Authors:** Alexis Fox, Junlin Wang, Paul Rosu, Bhuwan Dhingra
- **Institution:** (Duke University / inferred from Bhuwan Dhingra)
- **Published:** July 22, 2026
- **Link:** https://arxiv.org/abs/2607.20064
- **Code:** https://github.com/... (this https URL referenced in abstract)
- **Venue:** arXiv preprint

---

## Problem Statement (Plain English)

LLM agents are bad at long tasks — things that require hundreds or thousands of steps, like solving puzzles, exploring environments, or doing research. The core problem is context management: you can't fit a whole session's observations into the model's limited context window. Existing solutions either keep minimal info (losing important details) or dump everything in (making retrieval noisy and expensive). The paper asks: can we give agents a "programmatic memory" — letting them write code to search their own history efficiently?

## Methodology (Technical)

**PRO-LONG** is a minimal context management framework with these components:

1. **Complete structured interaction log:** Every observation, action, and result is saved in a structured format (JSON-like), preserving full history without truncation.

2. **Programmatic memory retrieval:** Instead of semantic search or embedding-based retrieval (which loses exact-match information), PRO-LONG capitalizes on recent advances in coding agents. The agent writes and executes search code against its own history — exact string matching, structured queries, regex, etc.

3. **Minimal context injection:** Only the search results (relevant history snippets) are loaded into the model's context, not the full history. The search code itself runs externally.

4. **No special architecture:** Works with any frontier LLM — no fine-tuning, no custom attention mechanisms. Just wraps the agent with a file-based memory system and code-execution access.

5. **Evaluation on ARC-AGI-3:** The full public game set of ARC-AGI-3 (a continual learning / reasoning benchmark with visual grid puzzles).

## Key Results (with numbers)

- **76.1% pass@1** on ARC-AGI-3 full public set (vs. base coding agent baseline)
- **97.4% best@2** with Claude Fable 5 at total cost of **$1,750**
- **18.0 percentage point average improvement** over base coding agents across frontier models
- **4.2-5.8x fewer tokens** consumed compared to state-of-the-art specialized harnesses
- Matches or exceeds specialized harnesses while being model-agnostic and simpler

## What's Novel

- **Programmatic memory** (agent writes code to search its history) is a new architectural pattern — prior work uses semantic search, summarization, or windowing
- Demonstrates that **exact retrieval > semantic retrieval** for long-horizon agent tasks
- The token efficiency gains (4-5x) are practically significant for cost-sensitive agent deployments
- Open-source code and logs available for reproducibility

## My Connection (to Manny's Work)

MCP servers and agent orchestration frameworks struggle with the same context-management problem PRO-LONG solves. If Manny is evaluating or building agent systems that operate over long sessions (recon agents, penetration testing agents, research assistants), the programmatic memory pattern is directly applicable — and the failure modes (search code errors, memory corruption, retrieval hallucinations) represent new attack surfaces for red-team testing.

## What I Learned (Plain English)

There's a simple but powerful idea here: instead of trying to fit a whole history into the model's context or using fuzzy search to find relevant bits, let the agent write Python to search its own history. An agent can use exact string matching to find what it needs — same way a human uses Ctrl+F. This avoids the "lost in the middle" problem (models forget things in the middle of long contexts) and is much more token-efficient. The trade-off is that the agent must be competent at writing search code, which not all models are. But as coding models improve, this pattern becomes increasingly viable.
