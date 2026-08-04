# Mutate to Bypass: Autonomous Endpoint Evasion via Knowledge-Driven Multi-Agent Orchestration

**Paper:** [arXiv:2608.01639](https://arxiv.org/abs/2608.01639)
**Authors:** Weifeng Yuan, Wenbo Guo, Qingyun Du, Jun Chen, Feng Dong, Haoyu Wang, Yang Liu
**Institution:** Nanyang Technological University, Singapore
**Published:** August 3, 2026
**Subjects:** Cryptography and Security (cs.CR); Operating Systems (cs.OS)
**Pages:** Not specified (475 KB, includes code)

---

## Problem Statement (Plain English)

There are many publicly documented EDR (Endpoint Detection and Response) evasion techniques, but it's unclear whether commercial EDR systems can actually defend against them. Testing requires turning fragmented knowledge (blog posts, PoCs, threat intel) into working, compiled payloads that bypass detection — a task that current automation doesn't handle. Manual reverse engineering is slow, incomplete, and skill-gated.

## Methodology (Technical)

AutoBypass is a knowledge-grounded, closed-loop multi-agent framework for automated EDR resilience assessment:

**1. Detection-Aware Knowledge Base (DAKB):**
Structures threat intelligence, expert analyses, and open-source PoCs into:
- Evasion techniques (API hooking, direct syscalls, unhooking, process injection variants, etc.)
- Operational constraints (what techniques work against which EDR platforms, required API calls, detection signatures)

**2. Multi-Agent Orchestration:**
- **Planning Agent:** Selects evasion techniques from DAKB based on target EDR profile
- **Code Generation Agent:** Generates polymorphic code implementing chosen techniques — mutating signatures, randomizing syscall sequences, modifying string literals
- **Compilation Agent:** Compiles payloads to executable binaries
- **Telemetry-Driven Reasoning Engine:** Analyzes EDR telemetry/alerts from failed attempts to diagnose what was detected and feeds corrective evidence back into the strategy

**3. Closed-Loop Process:**
Attack → Observe detection → Diagnose failure via telemetry → Update strategy → Generate new polymorphic variant→ Repeat until bypass succeeds or budget exhausted.

## Key Results

- **Tested against 7 commercial endpoint security platforms**
- **90% evasion rate against Windows Defender**
- **86.7% evasion rate against Trend Micro AV**
- **Bypassed every single one of the 7 target platforms** (100% of platforms had at least one successful bypass)
- **Knowledge base impact on small models:** 8B open-weight models went from 27-53% success rate to 43-83% success rate with DAKB integration
- **Small models with DAKB approached parity with large proprietary models** (GPT-5 class), demonstrating that structured knowledge compensates for model size

## What's Novel

1. **Closed-loop telemetry reasoning** — the system doesn't just try random mutations; it reads EDR alerts/telemetry to diagnose *what was detected* and *why*, then generates targeted countermeasures. This mirrors how a human red teamer iterates.
2. **Knowledge-grounded small models** — the DAKB lets 8B open-weight models approach proprietary frontier model performance, democratizing offensive capability away from GPT-5/Claude dependency.
3. **Polymorphic code generation** — not just template-filling but actual code mutation that produces structurally different binaries with the same evasion behavior, defeating signature-based detection.
4. **Real commercial EDR targets** — tested against deployed commercial products, not academic benchmarks, making the results operationally relevant.

## My Connection (to Manny's Work)

This is directly actionable for red team operations. AutoBypass operationalizes what a skilled EDR evasion specialist does manually — reading threat intel, understanding detection logic, iterating on payloads, reading alert telemetry, and mutating code — into an automated pipeline. The framework could be integrated into red team infrastructure to generate initial bypass payloads targeting specific EDR products, with the human operator validating and deploying the best candidates. The knowledge base structure (technique → constraints → applicable EDR) is also a useful model for organizing any red team's evasion knowledge. The closed-loop approach — where failure telemetry drives iteration — is the same pattern needed for any autonomous offensive agent targeting adaptive defenses.

## What I Learned (Plain English)

Multi-agent AI systems can now autonomously generate working malware that evades commercial EDR products — and they can do it reliably. The key trick is giving the AI agents a structured knowledge base of known evasion techniques and then letting them iterate: try, see what gets caught, figure out why, try a new variant. Small models (8B parameters) work nearly as well as GPT-5 when given this structured knowledge, which means the capability is cheap and accessible. Every commercial EDR tested had at least one successful bypass, and Windows Defender — the most widely deployed EDR — was evaded 90% of the time. The message for defenders: your EDR's evasion resistance needs continuous, automated testing, because your adversaries can now run that test for you.
