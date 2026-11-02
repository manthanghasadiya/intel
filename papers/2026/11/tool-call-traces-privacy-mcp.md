# How Much Can Tool Calls Reveal About You? Privacy Leakage from Tool-Call Traces (PrivacyTrace)

**Authors:** Donghang Duan, Qingyu Liu, Yifei Liu, Na Zou, Xia Hu, Dongrui Liu
**Affiliation:** Shanghai Artificial Intelligence Laboratory · University of Electronic Science and Technology of China · The University of Sydney
**Published:** October 30, 2026 · **Link:** https://arxiv.org/abs/2610.41885
**Code/data:** https://github.com/AI45Lab/PrivacyTrace

---

## Problem Statement

Every MCP tool call leaves a record with the server: which endpoint, which tool, which schema and docstring were shown, what arguments the agent sent, and what the tool returned. Security research on agents has focused on whether the agent *transmits* your data somewhere it shouldn't. This paper asks a different question: what can an observer *infer about you* from perfectly legitimate, protocol-valid tool calls — no malicious tool, no access to your conversation, no exfiltration?

That matters because MCP servers are third parties. A calendar MCP server, a shopping MCP server, a travel MCP server each see a slice of your behavior, and if traces can be linked across servers, they see a lot more than any one of them should.

## Methodology

**Threat model.** A latent user profile `P = (p1..pd)` of `d` attributes. The agent executes a task, producing a trajectory; from that the observer extracts an ordered tool-call trace `T = (c1..cn)` where each call `ci = (endpoint, tool name + visible docs/schema, arguments, response)`. Critically, the trace **excludes the user–agent dialogue and the model's reasoning** — only what a server-side observer actually sees.

**Benchmark construction.** 1,000 population-calibrated synthetic user profiles (population statistics for realism, LLM-based consistency checking for coherence), each contributing one trajectory in each of four domains → **4,000 trajectories, 67,950 tool calls, 17 target attributes per profile**, generated with real MCP tools from **25 servers** (68,000 view–attribute slots in single-domain evaluation). Synthetic users sidestep the consent/governance wall that blocks real private-label datasets.

**Scoring.** Average attack success rate over view–attribute pairs. Attributes use deterministic scoring rules first; a hybrid scorer falls back to GLM-5.2 for unresolved semantic cases — and that judge has no trace access. Unparseable generations stay in the denominator as failures. Reference condition: prior-based guessing, where the attacker only sees the attribute definitions.

**Ablations.** Trace access on/off; observer scope (single local server → single domain → four linked domains); visible call volume; four LLM attackers; channel ablation (metadata + call order → + arguments → + returned results → full trace); one-shot vs two-stage inference.

## Key Results

**Traces beat priors, meaningfully but not enormously.**
- Single-domain trace access raises ASR from **17.97% → 26.25%** (+8.28 points) under a forced-prediction protocol.
- Observer scope is the strongest amplifier: **24.24%** local-server views, **26.25%** single-domain, **32.77%** linked four-domain views. Paired comparison of local-server → single-domain shows a **+3.09 point** within-task gain, i.e. profile clues are *spread across servers* and a domain-wide observer recombines evidence no single server sees.
- More calls, more leakage: **20.87%** at 1–5 visible calls → **29.26%** at 21–40.

**Not attacker-specific.** Four LLMs land within **25.29%–28.32%** ASR:

| Attacker | ASR | Est. cost |
|---|---|---|
| MiniMax-M3 | 25.29% | $63.70 |
| DeepSeek-V4-Flash (default) | 26.25% | $46.49 |
| GPT-5.5 | 27.78% | $1,079.88 |
| Gemini-3-Flash | 28.32% | $155.13 |

A ~$46 attack performs within 2 points of a ~$1,080 one. This is cheap.

**What leaks is traits, not identifiers.** ASR reaches **57.73% for citizenship** and **50.03% for religious belief**, while only **35 of 16,000** synthetic identifier predictions were correct (34 names, 1 phone number, zero emails, zero government IDs). Traces infer who you *are*, not what your account number is — which is exactly the class of data most privacy frameworks don't instrument.

**Domain matters.** Single-domain ASR ranges from **21.57%** (career learning) to **31.86%** (shopping), with travel at **28.33%** — concrete locations, products, budgets and returned entities are what carry the signal.

**The channel ablation is the actionable finding.**
- Metadata + call order alone: **14.53%** — *below* schema-only prior guessing (17.97%). Call structure alone is worthless.
- Add arguments: **27.71%** (+13.18 points).
- Add returned results: **24.54%** (+10.01 points).
- Full trace: **26.25%** — 1.46 points *below* arguments-only, so results and arguments overlap rather than stack.

Arguments are the highest-value field: the entities, constraints, locations and queries the agent chooses to send already expose substantial profile information without any result bodies.

## What's Novel

1. Reframes MCP privacy from "did the agent leak data" to "what does a protocol-valid trace imply about the user" — no malicious tool required, so nothing in the tool-trust model catches it.
2. PrivacyTrace as a benchmark: 1,000 calibrated profiles, 4,000 real MCP trajectories over 25 servers, 17 attributes with explicit scoring rules — reusable, and the first of its kind for semantic tool records.
3. The observer-scope result (24% → 33% as servers link) quantifies the value of trace correlation, which is the argument for why "each server only sees a little" is not a defense.
4. Channel ablation that isolates *arguments* as the dominant leak vector.

## My Connection

- **MCP server operators are the observer.** If you run an MCP server, you're the attacker in this threat model — and the paper says the highest-value field you hold is the argument payload, not the response. Logging policy should treat arguments as PII, not as telemetry.
- **Red team framing.** This is a passive, non-invasive attack: no injection, no exploit, nothing anomalous in the protocol. It's the kind of finding that survives a "we reviewed our agent for prompt injection" security program untouched.
- **Cross-server linkage is the escalation path.** A single-server compromise is 24%; a domain-wide or multi-server position is 33%. For agents that fan out to many MCP servers, whoever sits in the middle gets the higher number — which is an argument for auditing aggregators and gateways first.
- **Detection.** Since metadata/order alone score below prior guessing (14.53%), don't build a "behavioral anomaly from call sequences" detector and expect privacy protection. The semantic content is the exposure.

## What I Learned

I assumed the dangerous part of an agent trace was the credentials and file paths in tool *results*. It's actually the *arguments* — the query the agent composes is a fairly honest statement of what the user wants, and wanting things is a demographic signal. Shopping and travel leak the most because they're full of concrete places, products and budgets.

The second lesson is about aggregation. Any single MCP server only sees a fragment, and each operator can reasonably say "I only see a little." But the numbers move from 24% to 33% the moment those fragments get linked, and combining fragments is exactly what an agent gateway does by design. The privacy risk isn't in any one server — it's in the architecture that routes everything through one place.
