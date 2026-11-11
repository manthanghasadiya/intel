# From Attack Surface to Defense Architecture: Compositional Security for Agentic Coding Harnesses

**Authors:** Simon Hegelich, Bardh Prenkaj, Habiba Sarhan, Morteza Shahrezaye, Kolja Hegelich, Christian Montag, Ralf Schneider, Erik Cambria
**Published:** 2026-11-10 (arXiv v1)
**Paper:** https://arxiv.org/abs/2611.13960
**Code:** Not named in the metadata; 21 pages, 3 figures + supplement
**Institution:** Not stated in the arXiv metadata (author affiliations span German and Italian institutions; confirm on the PDF)

---

## Problem Statement

A coding agent is not a chatbot with a shell. Once it can run `bash`, `curl`, `find` and `wget`, the harness itself becomes an automated living-off-the-land (LotL) platform: the attacker never needs to drop a binary, because the agent will happily use the binaries already installed. Prompt injection into retrieved content — a README, a dependency file, a scraped doc — turns dual-use tooling into the payload.

The paper's core claim is architectural: **security in agentic systems is a structural property of the cognitive architecture and the composition of permissions, not an emergent property of the model.** Two consequences follow immediately:

1. Regex blacklists over generated commands miss structural evasions (the same action expressible through a different binary, a wrapper, or a chain).
2. LLM-as-a-Judge guardrails are both slow and badly calibrated — they trade latency for over-blocking benign work, which is the failure mode that gets a security control switched off in production.

## Methodology

They build **ActiveSecHarness**, a neurosymbolic reference monitor around a coding agent, organized by one principle: **Separation of Intelligence and Authority**. The model can reason about anything, but it cannot *authorize* anything — authority is granted by tokens, checked deterministically.

Four components:

1. **Capability tokens.** Macaroon-inspired HMAC tokens distributed across four **Trust and Authority Rings**. Capabilities are attenuated — a token handed to a sub-agent can only be narrower than the one the parent held. This gives you delegation with cryptographic proof of scope rather than a boolean "the agent is allowed to run commands."
2. **2D product taint lattice.** `L = L_trust × L_sens` — trust level crossed with data sensitivity — plus a **workspace provenance ledger** that tracks where every artifact came from. The product lattice is what stops *secret laundering*: a secret read from `.env` and written into a build log crosses a sensitivity boundary even if it never leaves the disk.
3. **Deterministic Ring-0 OS/tool guards.** Checks at the syscall/tool boundary, no model in the loop, no natural-language policy interpretation.
4. **Supporting machinery:** bytecode certificates (what was actually approved is what actually runs), split-horizon audit logs (the agent cannot see or edit the log it is being judged by), self-healing security test synthesis, and an adaptive `UserWikiManager` that accumulates per-workspace policy context.

Evaluation: **N = 252 fixtures** — 120 attack, 120 benign, 12 multi-step kill chains.

## Key Results

| Configuration | Attack Block Rate (ABR) | False Positive Rate (FPR) | Latency |
|---|---|---|---|
| Ring 0 deterministic guards | **1.000** | **0.000** | **12.7 µs** |
| Local LLM judge (qwen3:4b) | — | 98.3% of benign work refused | 142.3 ms |

- Ring 0 is **11,200× faster** than the LLM judge and does not degrade benign workflows at all.
- Multi-step trajectory ablation, 12 kill chains:
  - Oracle "Conseca-style" stateless policy: **misses 4/12** (ABR = 0.667).
  - Coarse Boolean taint: ABR improves but **blocks 83.3% of benign workflows** — unusable.
  - Their 2D product lattice: **ABR = 1.000, FPR = 0.000**, ~20 µs per step.

The interesting number is not "1.000 ABR" — fixtures are fixtures. It is that the *cheap* deterministic check beats the expensive semantic one on both axes simultaneously, and that the 2D lattice is what makes determinism viable without destroying utility.

## What's Novel

- **Framing:** the harness as an automated LotL platform. This is the correct abstraction and it reframes harness hardening as permission composition instead of prompt defense.
- **Separation of Intelligence and Authority** as an explicit design rule: the model proposes, tokens dispose. Widely implied in the literature, rarely stated as an invariant with a reference monitor behind it.
- **The 2D product lattice (trust × sensitivity) with provenance.** Most taint work in agents is 1D (tainted/untainted), which is why it over-blocks. Adding an orthogonal trust axis and a disk-level provenance ledger is what buys FPR = 0.
- **Split-horizon audit logs** — modeling the agent as a potential adversary *against its own telemetry*, which almost no harness does.
- A quantitative head-to-head against LLM-as-a-Judge guardrails on the same fixtures, which is the comparison practitioners actually need.

## My Connection

This is a harness-security architecture paper, which is Manny's working surface:

- **For agent security research:** the 12 kill-chain ablation is a ready-made evaluation protocol. If you are measuring whether a harness stops multi-step attacks, "oracle stateless policy misses 1 in 3 chains" is the baseline you have to beat, and it is a much fairer baseline than "no defense."
- **For red teaming:** the paper implicitly enumerates the ring boundaries you would attack. Capability-token attenuation bugs, provenance-ledger gaps across disk, and the `UserWikiManager`'s learned policy context are all new state — and new state is new attack surface. The ledger is a *record of what the agent did*; anything an attacker can do to the ledger's fidelity is a detection-evasion primitive.
- **For tooling:** Macaroon-style attenuated capability tokens are implementable today in MCP (HMAC + caveats), and they compose better with MCP's tool-scoped model than coarse OAuth scopes. This is the most directly portable idea in the paper.
- **Caveat worth carrying into any citation:** 252 hand-built fixtures, one local judge model, one harness. ABR = 1.000 on a fixed fixture set is a claim about that set.

## What I Learned

You cannot make an agent safe by making it smarter about safety. The model that decides "is this command dangerous?" is the same model that an attacker just injected instructions into — and asking it to judge itself costs 142 ms per decision while refusing almost everything legitimate. What actually works is boring: give the agent a token that says exactly what it may touch, check that token at the syscall boundary in microseconds, and keep a tamper-resistant record of where every file came from so a secret can't be walked out one hop at a time. The clever part isn't the defense; it's the bookkeeping — two independent axes of risk instead of one, plus provenance on disk — which is what lets a dumb, fast check be strict without being useless.
