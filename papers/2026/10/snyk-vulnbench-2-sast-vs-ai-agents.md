# How Deterministic SAST and AI Security Agents Divide the Work: Snyk VulnBench 2.0

**Authors:** Liran Tal, Chris Liu, Charlotte Fereday, Sebastian Roth, Brian Clark
**Institution:** Snyk (authors are Snyk staff)
**Published:** October 28, 2026
**Link:** https://arxiv.org/abs/2610.38617
**Code / data:** Benchmark harness, reference sets and all run data are public (links in the paper); companion site referenced. 44 pages, 21 figures, 7 tables.

---

## Problem Statement

Everyone now assumes AI security agents find bugs that static analysis misses. Almost nobody has measured the split cleanly, because the two tool classes fail in different ways and most benchmarks reward whichever one reports more findings.

VulnBench 2.0 asks two precise questions:
1. What do AI coding agents and AI security harnesses find that deterministic SAST does not?
2. What do they **miss** that SAST catches?

The scoring is deliberately hostile to volume-based cheating. A finding only counts if it locates **both** the attacker-controlled source **and** the dangerous sink, **each within two lines**. Reporting "this file looks sketchy" scores zero.

## Methodology

**Corpus:** 108 hand-curated, attacker-reachable vulnerabilities across 20 open-source web applications in Go, JavaScript/TypeScript, Python and Java. 62 of the 108 flows cross more than one file — so the benchmark requires interprocedural reasoning, not pattern matching.

**Configurations evaluated (nine, under one blind protocol):**
- Snyk Code — the deterministic static analyzer, as the baseline.
- Claude Code `/security-review` across three model/effort settings, including one wired to the **Snyk MCP server** — i.e. the AI agent given the SAST tool as a tool.
- Codex Security with GPT-5.6 Luna, Terra and Sol.
- DeepSec scanner with two model backends.

**Harness ablations** are the interesting part: they run the *same model* with and without built-in review policy, and they measure file-level versus line-level localization separately. That separates "which model" from "which scaffolding," which almost no benchmark does.

## Key Results

| Configuration | Result |
|---|---|
| Snyk Code (SAST) | 82.4% macro recall — 89 of 108 |
| Claude Code + Opus 5, xhigh effort | 79.3% — 85 of 108 |
| Union of the two | **101 of 108** |

- The top two are **statistically indistinguishable**. They disagree on 28 vulnerabilities, split 16 to 12 — too few to separate them.
- Neither is sufficient alone. Together they reach 101/108, which is the ceiling of all seven comparable configurations combined.
- The AI configurations also reported **at least seven real vulnerabilities** that were not in the reference set and that Snyk Code did not report — so the 101 is a floor for the union, not a ceiling for AI.
- **Harness beats model.** Removing Claude Code's built-in review policy dropped the same model from matching 16 to 9 of the 20 vulnerabilities in the classes that policy covers or restricts (supplementary run on 19 of 20 apps). The policy, not the weights, was doing half the work.
- **Localization vs detection is a real gap.** Codex Security with GPT-5.6 Sol located 80 vulnerabilities at file level but only 34 at line level — it knows *where to look* far better than *where the bug is*.
- **Model-free file selection is a bottleneck.** DeepSec's non-model file triage surfaced a file from only 56 of the 108 flows — nearly half the corpus never got in front of the model.
- Coverage and cost were only loosely related.

## What's Novel

Three things this benchmark does that most don't:

1. **Symmetric scoring.** It penalizes AI agents for what SAST catches and SAST for what agents catch. Most comparisons quietly only count the second direction.
2. **Source-and-sink, two-line precision.** This is close to how a real reviewer validates a finding, and it correctly destroys the "flagged the file, call it a hit" inflation that plagues LLM security evals.
3. **Harness ablation as a first-class variable.** Isolating review policy from model, and file-level from line-level localization, gives an actual engineering answer to "should I switch models or fix my scaffolding?" The answer here is scaffolding, consistently.

The "AI + SAST union beats either" result is the practical headline, and the seven extra real vulnerabilities the AI found outside the reference set is the strongest evidence in the paper that the reference set itself is incomplete.

## My Connection

This is the most useful defensive benchmark I've seen for Manny's workflow, and it's also an offensive calibration tool.

1. **Know what your target's scanner can't see.** If an org relies on Snyk Code alone, 19 of 108 attacker-reachable flows are invisible to it. If they rely on an AI agent alone, roughly 23 are. The blind spots differ, and the disagreement set of 28 is a target list.

2. **It reframes "AI finds more bugs" claims.** Today's Gemini 4 Carbon announcement cites SEC-bench Pro 88.2% and CWE-bench 71.0%. VulnBench says: fine, but which harness, at what localization precision, and what did it miss that a deterministic scanner caught? Ask for the union number, not the model number.

3. **Line-level precision is the metric to quote.** 80 file-level vs 34 line-level for the same model is the concrete counter to "the agent found 80 vulnerabilities." Triage cost lives in the 46-item gap.

4. **The Snyk MCP server configuration is the pattern to study.** An AI agent plus a deterministic tool exposed over MCP outperformed either alone. That's the correct architecture for agentic security tooling — deterministic scanners as MCP tools, model as the reasoning layer, not the model as the scanner.

5. **Cost and coverage are decoupled**, which kills the "just spend more tokens" instinct.

## What I Learned

SAST and AI agents aren't competing implementations of the same function. They have near-identical recall and *different* blind spots, which means the correct move is to run both and treat their disagreement as the signal.

The bigger lesson is that the scaffolding around a model contributes about as much as the model choice. Stripping Claude Code's review policy halved its performance in exactly the vulnerability classes that policy targets. When someone tells you their agent is good at security, the question isn't which model — it's which harness, and can I see the harness.
