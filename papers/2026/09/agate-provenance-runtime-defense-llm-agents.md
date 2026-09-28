# AGATE: Provenance-Based Runtime Defense Against Compositional Attacks on LLM Agents

**Authors:** Xiaorui Zhang, Zhuoran Cheng, Kailin Liu, Zhaoxi Sun, Shiyu Fan, Tongyu Yuan, Bin Yuan, Weizhong Qiang, Deqing Zou
**Institution:** Huazhong University of Science and Technology and collaborators
**Date:** Submitted Sep 25, 2026 · cs.CR, cs.AI
**Link:** https://arxiv.org/abs/2609.30830
**Code:** Adapters for DeepSeek Harness, OpenCode and OpenClaw (host code unmodified)

---

## Problem Statement (plain English)

An agent rarely does one obviously bad thing. It does a hundred ordinary things — read a file, fetch a URL, summarise, write a file, call an API — and the *sequence* is what's harmful. No single step looks like an attack, so per-action allow/deny lists miss it entirely.

To judge a sequence you need two things about every action:

- **Authority** — is this operation permitted at all, and under what grant?
- **Provenance** — where did the data being moved actually come from?

Most current agent defenses supply neither rigorously. They rely on static permission rules (which can't anticipate open-ended tasks) or an LLM judging each action (which is inconsistent and itself injectable).

## Methodology (technical)

AGATE is an **authorization and data-provenance gate** placed at instrumented agent-harness boundaries. Key design points:

- **Authorization grounding:** operator declarations plus host approval events establish what's authorized. **Delegated actions are constrained by grants that bind to exact parameters, expire, and permit a limited number of uses** — so a grant for one call can't be replayed or broadened.
- **Source registration** connects observed inputs to subsequent transfers — this is the provenance half. Data carries its origin forward, so a value that came from an untrusted fetch is tagged when it later flows into a write.
- **Effect ledger** tracks repeated requests, catching slow-burn/looping abuse that any single-action check would let through.
- **Deterministic checks, no LLM in the decision path.** Decisions are computed, not judged, and each retains its grounds plus execution evidence for **forensic replay**.
- **Three production harness adapters** — **DeepSeek Harness, OpenCode, OpenClaw** — integrated **without modifying host code**, translating each host's native observation and veto points into one shared gate interface. The judgment core is identical across all three; only enforcement depth varies.

Evaluation combines **153 exercised attack-chain records** with deployment, utility and reconstruction experiments.

## Key Results (with numbers)

- **Deployment observations** revealed how tool declarations and data checks govern business actions — including a **bypass via parameter rewriting**, i.e. an agent (or injection) achieving the same effect by re-shaping the arguments rather than changing the call.
- **Utility cost is real and measured:** **6 of 11 benign file-processing scenarios produced denial events**, exposing the price of content-based provenance policies — legitimate workflows get blocked.
- **Forensic reconstruction holds up:** across **252 runs on 63 sanitized scenarios**, replay **agreed with live graph projections for all 63 scenarios on each of two platforms** — the audit trail is faithful, not approximate.
- The paper is candid about **concrete limits**: **content transformation** (data laundered through reformatting defeats naive provenance), **legitimate reuse** (the same value genuinely used twice), and **observation coverage** (you can only reason about what the harness exposes).

## What's Novel

- **Provenance + authorization in one gate at the harness boundary.** Most work does one or the other; compositional attacks need both, because authority without provenance can't see tainted data and provenance without authority can't see unauthorized effect.
- **Grants that bind to exact parameters, expire, and have use counts.** This is a capability-style model applied to tool calls — a meaningful upgrade over coarse allow/deny rules and over per-action LLM judgment.
- **Zero-LLM decision path with replayable evidence.** Directly answers the criticism that LLM-based guards are themselves attack surface and produce inconsistent verdicts.
- **Three real harnesses adapted without patching host code** — a deployment-realistic claim that most papers don't attempt.
- **Negative results published:** a parameter-rewrite bypass and a measurable false-denial rate. Publishing your own bypass is a strong signal of an honest evaluation.

## My Connection (to Manny's work)

This is the defense-side counterpart to today's offense paper (AgentXploit, 2609.31318) and it's operationally testable:

- **OpenClaw is in scope.** The adapter set includes OpenClaw, which sits in the same harness family Manny already works with — AGATE's gate is directly exercisable against it.
- **The parameter-rewrite bypass is a red-team primitive.** For any engagement where the client has deployed an agent gate, the test is: hold the intent constant, morph the arguments, and see whether authorization still fires. AGATE documents that its own content-based checks can be beaten this way — a concrete finding to reproduce.
- **Deterministic gates are the emerging consensus.** AGATE, NVIDIA's OpenShell policy prover, and prior work like ActGov (2609.24446) all converge on the same thesis: **keep the LLM out of the allow/deny decision.** That's now the design pattern to recommend in hardening engagements.
- **Forensic replay matters for reporting.** AGATE's replay-matches-live result (63/63 on two platforms) is the kind of evidence you need to make an agent's actions admissible in an incident report.

## What I Learned (plain English)

Two things stuck. First, **the utility cost is unavoidable and quantified** — 6 of 11 benign file tasks hit denial events. Tight agent provenance gates *will* break legitimate work, and anyone promising frictionless "zero trust for agents" is selling something they haven't measured. Second, the failure mode isn't the rule, it's the *representation*: an attacker who keeps the intent and rewrites the parameters can slip past content-based checks, which means any gate must reason about effects and origins, not about strings. And the quiet win is architectural — putting the gate at the harness boundary, deterministically, with replayable evidence, means you can audit the agent *after* it misbehaves without trusting the agent's own logs. One sentence: *provenance tells you where data came from, authorization tells you whether the action was allowed, and you need both, outside the model, or composition beats you every time.*
