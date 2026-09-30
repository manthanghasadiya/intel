# ToolFence: Fine-Grained Authorization for Secure Tool-Using LLM Agents

**Authors:** Yanjie Li, Xiangyu He, Xuelong Dai, Bin Xiao
**Institution:** (Hong Kong Polytechnic University — per author affiliation conventions)
**Published:** 2026-09-29
**Link:** https://arxiv.org/abs/2609.37196
**Code:** Not stated in abstract

---

## Problem Statement (plain English)

Tool-using LLM agents are vulnerable to **indirect prompt injection**: because trusted instructions and untrusted observations (web pages, files, tool outputs) share one context, malicious content can steer the agent — and the *defenses meant to stop it*. Existing input-filtering defenses fail because a **within-tool attack** keeps the *intended* tool but manipulates its *arguments*; content inspection can't tell a user-authorized argument from an attacker-supplied one. Multi-path consensus defenses leave a high attack success rate because they examine content or aggregated outputs rather than **authorizing effects**. Stronger data-flow-control defenses (e.g., CaMeL) give better guarantees but add so much latency they're impractical. The paper's goal: authorize *effects* with provenance at practical speed.

## Methodology (technical)

ToolFence works in three moves:

1. **Typed authorization blueprint (pre-execution).** Before the agent runs, ToolFence compiles a blueprint that declares, per tool, which effects and argument values are in scope — a *typed* contract, not free text.
2. **Deterministic monitor (fast path).** A low-overhead monitor enforces the blueprint at call time. Because it is deterministic, most calls are adjudicated without any model in the loop.
3. **Lazy capability grants (slow path).** When the blueprint is *incomplete*, ToolFence doesn't adjudicate each concrete call with an LLM judge. Instead it asks a **judge to grant a new capability**, which is then enforced deterministically on subsequent calls.

The key design property is **fine-grained, provenance-aware authorization**: values that trace back to the user are treated differently from values that came from untrusted observations, which is precisely what defeats the within-tool attack (attacker-chosen arguments in an otherwise legitimate call).

## Key Results (with numbers)

- **On AgentDojo with Qwen3-max:** overall attack success rate (ASR) reduced to **near zero** (effectively 0 per the abstract's "near zero").
- **Clean-utility cost:** only a **3.80 percentage-point drop**.
- **Runtime:** "practical runtime overhead" — the deterministic fast path plus capability-level grants **substantially reduce expensive judge calls**.
- **Baselines beaten:** input-filtering defenses, multi-path consensus (content/aggregate-output based), and CaMeL-style data-flow control (the latter on latency grounds).

## What's Novel

- Shifts the defense target from **content** to **effects** with explicit **provenance**, which is the only framing that cleanly separates trusted and untrusted values in a shared context.
- **Compiles** an authorization contract ahead of execution rather than adjudicating at runtime — moving security out of the hot path.
- The **lazy capability-grant** pattern (grant a capability once, then enforce deterministically) is an efficiency idea borrowed from OS capability systems, adapted to agent tool calls.

## My Connection (to Manny's work)

This is the strongest "authorize effects, not text" artifact to cite against the wave of "just filter the retrieved content" defenses. It directly addresses **within-tool attacks** — the same class Manny uses to demonstrate that an agent with the right tool and the wrong argument is a compromised agent. It also pairs naturally with today's MCP RCE story (CVE-2026-102911): a blueprint that pins the `url` argument's provenance would have neutralized that sink.

## What I Learned (plain English)

You cannot secure an agent by reading its inputs, because the dangerous part is often a *legitimate call with attacker-chosen arguments*. The workable fix is to declare, before the agent runs, what each tool is *allowed to do and with whose values*, then enforce that contract with cheap deterministic code and only escalate to a model when the contract has a gap.
